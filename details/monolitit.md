## Probleme

Des fichiers de taches de roles melangent de nombreuses taches indépendantes dans un seul fichier. Cela les rend :

- **Difficiles a relire** — une PR touchant une section imacte le fichier entier
- **Difficiles a reutiliser** — on ne peut pas inclure uniquement la section "utilisateur admin" sans tout executer
- **Difficiles a tester** — pas moyen d'executer des sous-ensembles du role de maniere isolee
- **Difficiles a maintenir** — trouver une tache specifique necessite de faire defiler des centaines de lignes

Aucun de ces fichiers n'utilise `import_tasks` ou `include_tasks` pour organiser leur contenu.

## Fichier 1 : `roles/cephadm/tasks/main.yml` (412 lignes)

### Structure actuelle

Ce fichier gere l'ensemble du cycle de vie du cluster Ceph dans un seul fichier : configuration des utilisateurs, installation des binaires, detection du cluster, configuration du registre, bootstrap, distribution des identifiants, deploiement des moniteurs, deploiement des OSD et configuration des pools.

### Decoupage propose

| Section | Fichier suggere | Description |
|---------|-----------------|-------------|
| Configuration utilisateurs et groupes | `setup_users_groups.yml` | Creer l'utilisateur cephadm, le groupe, le compte de service (specifique a la distribution) |
| Installation binaires et paquets | `install_cephadm.yml` | Telecharger, installer, configurer le binaire cephadm, le depot, ceph-common |
| Detection du statut du cluster | `detect_cluster_status.yml` | Detecter le cluster existant, classifier les noeuds, definir les variables de bootstrap |
| Registre de conteneurs local | `setup_local_registry.yml` | Configurer le registre non securise, pull/tag/push des images Docker |
| Bootstrap du cluster Ceph | `bootstrap_cluster.yml` | Generer ceph.conf, bootstrapper le nouveau cluster (conditionnel) |
| Distribution des identifiants | `distribute_credentials.yml` | Recuperer/distribuer les cles SSH cephadm pour l'acces sans mot de passe |
| Ajout des noeuds moniteur | `add_monitors.yml` | Ajouter les moniteurs a l'orchestrateur, verifier le monmap |
| Deploiement des daemons OSD | `deploy_osds.yml` | Detecter les OSD existants, nettoyer les volumes, appliquer la spec OSD |
| Gestion des pools et utilisateurs | `configure_pools_users.yml` | Verifier la sante, creer le pool RBD, creer/mettre a jour client.libvirt |
| Nettoyage | `cleanup_registry.yml` | Arreter et supprimer le conteneur de registre Podman |

### `main.yml` resultant

```yaml
---
- name: Configuration des utilisateurs et groupes
  ansible.builtin.import_tasks: setup_users_groups.yml

- name: Installation de cephadm
  ansible.builtin.import_tasks: install_cephadm.yml

- name: Detection du statut du cluster
  ansible.builtin.import_tasks: detect_cluster_status.yml

- name: Configuration du registre de conteneurs local
  ansible.builtin.import_tasks: setup_local_registry.yml

- name: Bootstrap du cluster Ceph
  ansible.builtin.import_tasks: bootstrap_cluster.yml

- name: Distribution des identifiants
  ansible.builtin.import_tasks: distribute_credentials.yml

- name: Ajout des noeuds moniteur
  ansible.builtin.import_tasks: add_monitors.yml

- name: Deploiement des daemons OSD
  ansible.builtin.import_tasks: deploy_osds.yml

- name: Configuration des pools et utilisateurs
  ansible.builtin.import_tasks: configure_pools_users.yml

- name: Nettoyage du registre
  ansible.builtin.import_tasks: cleanup_registry.yml
```

---

## Fichier 2 : `roles/debian_hardening/tasks/main.yml` (384 lignes)

### Structure actuelle

Ce fichier implemente le durcissement ANSSI BP-28 dans un seul fichier monolithique. La plupart des sections suivent un double pattern : "appliquer le durcissement" + "annuler le durcissement" controle par une variable `revert`.

### Decoupage propose

| Section | Fichier suggere | Description |
|---------|-----------------|-------------|
| Creation de groupes | `setup_groups.yml` | Creer les groupes ansible et privileged |
| Parametres noyau | `configure_kernel_params.yml` | Ajouter/annuler les parametres noyau via GRUB |
| Durcissement sysctl | `configure_sysctl.yml` | Core dumps, kexec, binfmt_misc, durcissement general |
| Environnement shell | `configure_shell_security.yml` | TMPDIR et timeout bash (scripts profile.d) |
| Mot de passe root aleatoire | `configure_random_root_passwd.yml` | Installer/gerer random-root-passwd.service |
| Durcissement SSH et sudo | `harden_ssh_sudo.yml` | Regles d'audit SSH, regles sudo (ANSSI BP-28) |
| Escalade de privileges | `configure_privilege_escalation.yml` | Groupe privileged, permissions binaire sudo |
| Connexion et PAM | `configure_login_pam.yml` | login.defs, desactiver su, securetty, pam_securetty |
| Durcissement systemd | `harden_systemd_services.yml` | Repertoires service.d, durcissement par service |
| Securite GRUB | `configure_grub_security.yml` | Mot de passe GRUB, classe de demarrage non restreinte |
| Configuration d'audit | `configure_audit.yml` | syslog.conf auditd, audit.rules, mise a jour GRUB |

### `main.yml` resultant

```yaml
---
- name: Configuration des groupes de durcissement
  ansible.builtin.import_tasks: setup_groups.yml

- name: Configuration des parametres noyau
  ansible.builtin.import_tasks: configure_kernel_params.yml

- name: Application du durcissement sysctl
  ansible.builtin.import_tasks: configure_sysctl.yml

- name: Configuration de la securite shell
  ansible.builtin.import_tasks: configure_shell_security.yml

- name: Configuration du mot de passe root aleatoire
  ansible.builtin.import_tasks: configure_random_root_passwd.yml

- name: Durcissement SSH et sudo
  ansible.builtin.import_tasks: harden_ssh_sudo.yml

- name: Configuration de l'escalade de privileges
  ansible.builtin.import_tasks: configure_privilege_escalation.yml

- name: Configuration de la connexion et PAM
  ansible.builtin.import_tasks: configure_login_pam.yml

- name: Durcissement des services systemd
  ansible.builtin.import_tasks: harden_systemd_services.yml

- name: Configuration de la securite GRUB
  ansible.builtin.import_tasks: configure_grub_security.yml

- name: Configuration de l'audit
  ansible.builtin.import_tasks: configure_audit.yml
```

---

## Fichier 3 : `roles/debian/tasks/main.yml` (291 lignes)

### Structure actuelle

Ce fichier gere la configuration post-installation Debian : vim, journalisation, utilisateur admin, chargeur de demarrage, depots, SSH. Un melange de taches ponctuelles et de groupes lies.

### Decoupage propose

| Section | Fichier suggere | Description |
|---------|-----------------|-------------|
| Configuration vim | `configure_vim.yml` | Defaults vim et coloration syntaxique |
| Configuration syslog-ng | `configure_syslog_ng.yml` | Repertoire de logs, config, certificats TLS |
| Detection ancien admin | `detect_old_admin.yml` | Detecter et supprimer l'ancien utilisateur admin de l'ISO |
| Journald | `configure_journald.yml` | Copier journald.conf, redemarrer le service |
| Utilisateur admin et sudoers | `setup_admin_user.yml` | Groupe, utilisateur, cles SSH, regles sudoers |
| Regles sysctl | `configure_sysctl.yml` | Copier la config sysctl panic/reboot |
| Environnement et shell | `setup_environment.yml` | /etc/environment, PATH et parametres .bashrc |
| Depots APT | `configure_apt_repos.yml` | Detecter et configurer les depots apt |
| Configuration GRUB | `configure_grub.yml` | Nettoyage FAI, parametres cmdline, OS prober |
| Configuration SSH | `configure_ssh.yml` | Desactiver PerSourcePenalties (Trixie) |

### `main.yml` resultant

```yaml
---
- name: Configuration de vim
  ansible.builtin.import_tasks: configure_vim.yml

- name: Configuration de syslog-ng
  ansible.builtin.import_tasks: configure_syslog_ng.yml

- name: Detection et suppression de l'ancien utilisateur admin
  ansible.builtin.import_tasks: detect_old_admin.yml

- name: Configuration de journald
  ansible.builtin.import_tasks: configure_journald.yml

- name: Configuration de l'utilisateur admin
  ansible.builtin.import_tasks: setup_admin_user.yml

- name: Configuration des regles sysctl
  ansible.builtin.import_tasks: configure_sysctl.yml

- name: Configuration de l'environnement
  ansible.builtin.import_tasks: setup_environment.yml

- name: Configuration des depots APT
  ansible.builtin.import_tasks: configure_apt_repos.yml

- name: Configuration de GRUB
  ansible.builtin.import_tasks: configure_grub.yml

- name: Configuration de SSH
  ansible.builtin.import_tasks: configure_ssh.yml
```

Note : Plusieurs de ces sous-fichiers de taches (`configure_vim.yml`, `configure_syslog_ng.yml`, `configure_journald.yml`, `setup_admin_user.yml`, `configure_sysctl.yml`, `setup_environment.yml`) sont identiques entre debian/centos/oraclelinux et devraient etre partages (voir AUDIT_DUPLICATE_CODE_FR.md).

---

## Notes d'implementation

### `import_tasks` vs `include_tasks`

- Utiliser **`import_tasks`** (statique) pour les inclusions inconditionnelles — elles sont resolues a l'analyse, supportent les tags et se comportent de maniere identique aux taches en ligne
- Utiliser **`include_tasks`** (dynamique) uniquement lorsque l'inclusion est conditionnelle (`when:`) ou que le nom de fichier est une variable


### Emplacement des fichiers

Tous les sous-fichiers de taches vont dans le meme repertoire `tasks/` que `main.yml` :

```
roles/cephadm/tasks/
├── main.yml                    # 30 lignes d'import_tasks
├── setup_users_groups.yml
├── install_cephadm.yml
├── detect_cluster_status.yml
├── setup_local_registry.yml
├── bootstrap_cluster.yml
├── distribute_credentials.yml
├── add_monitors.yml
├── deploy_osds.yml
├── configure_pools_users.yml
└── cleanup_registry.yml
```
### Extras
Certaines taches de configurations des OS comportent de la duplication de code: see: https://github.com/AntoineDupre/seapath-ansible/wiki/Code-dupliqu%C3%A9

Il est difficile de tester les fichiers monolythiques, il est important de les restructurer pour pouvoir ecrire des tests simple. Un exemple de test et de restructuration peut etre vu [ici](https://github.com/AntoineDupre/seapath-ansible/tree/refactor-centos-roles/roles/centos)


