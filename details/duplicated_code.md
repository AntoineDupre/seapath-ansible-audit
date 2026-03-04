 Extraire le code duplique dans des roles/fichiers de taches partages

## Probleme

Du code est dupliqué massivement entre les roles specifiques aux distributions et les roles de machines physiques. Des blocs de taches identiques sont copies-colles entre les variantes `debian`, `centos` et `oraclelinux`. Cela signifie :

- **Les corrections de bugs doivent etre appliquees 3 fois** — une par distribution
- **La divergence est inevitable** — les copies derivent au fil du temps (et c'est deja le cas)
- **La charge de revue est triplee** — les PRs touchant une distribution devraient toucher les trois

## Remarque sur la structure

Le but de ces roles est de configurer la distribution. Le role est composé de taches qui peuvent etre regroupé en thématique. Au lieu d'avoir un seul gros fichier `tasks/main.yml` il est préférable de sépar r chaque outils que l'on souhaite configurer en sous taches. Cela permet:
 - une meilleur granularité lors de l'installation
 - ajouter des tags spécifiques pour chaque taches (exemple `vim` `grub`)
 - regrouper explicitement les taches qui concerne un meme outils
 - permettre une meilleur testabilité (il est difficile de tester l'ensemble d'un monolithe).

Ainsi il est possible de structurer les taches:
- tasks/main.yml (appel les autres taches)
- tasks/grub.yml
- tasks/logging.yml
- tasks/network.yml
- tasks/sysctl.yml
- tasks/users.yml
- tasks/vim.yml

Un exemple de restructuration du role `roles/centos` est disponible dans [cette branche](https://github.com/AntoineDupre/seapath-ansible/tree/refactor-centos-roles/roles/centos) qui ajoute des tests molecule pour le role.

## Zones de duplication

### 1. Roles de configuration des distributions (`debian`, `centos`, `oraclelinux`)

#### Taches Identiques

**Configuration vim**:  desactiver les defaults vim, activer la coloration syntaxique, supprimer vimrc.local. La seule difference est le chemin (`/etc/vim/vimrc` sous Debian vs `/etc/vimrc` sous CentOS/OracleLinux), qui peut etre gere avec une variable.

**Configuration syslog-ng**: creer le dossier de logs, copier la config syslog-ng, bloc complet de certificats TLS (creer les repertoires cert/ca, copier les certificats/cle/CA). Identique dans les trois distributions.

**Configuration journald**: copier journald.conf, redemarrer journald. Identique dans les trois.


**Regles sysctl:** copier `00-panicreboot.conf`. Identique (difference mineure dans le format `src:` de CentOS).

**Personnalisation de l'environnement**: personnaliser `/etc/environment` avec EDITOR, TERM, HISTSIZE, PATH, LIBVIRT_DEFAULT_URI. Identique.

**PATH pour l'utilisateur admin**: ajouter le PATH au `.bashrc` de l'utilisateur admin. Identique.

#### Taches partiellement identiques

**Creation de l'utilisateur admin**: creer le groupe admin, ajouter l'utilisateur avec mot de passe, ajouter les cles SSH, installer les regles sudoers, supprimer l'ancienne ligne sudoers. Identique.

Note : Debian et OracleLinux partagent egalement un bloc supplementaire de "detection et suppression de l'ancien utilisateur admin" que CentOS n'a pas.


**Configuration GRUB**: taches de parametres GRUB de base sont identiques. Differences :
- Debian a des taches supplementaires de nettoyage FAI
- CentOS/OracleLinux ont la desactivation de NetworkManager + activation de systemd-networkd a la fin
- La commande de mise a jour GRUB differe (`update-grub` vs `grub2-mkconfig`)



### 2. Roles de machines physiques (`debian_physical_machine`, `centos_physical_machine`, `oraclelinux_physical_machine`)

**Regles sysctl**: Copier bridge_nf_call.conf, ajouter le sysctl supplementaire depuis l'inventaire. Identique.

**Configuration commune (dossier src, vm_manager, python3-setup-ovs)**: Creer le dossier src, deployer vm_manager, deployer python3-setup-ovs, creer le repertoire pacemaker RA. Quasi identique, differences conditionnelles mineures.

**chrony-wait.service**: Copier et activer chrony-wait.service. Identique.

**Configuration du service Pacemaker**: Creer pacemaker.service.d, copier le drop-in, verifier le statut, desactiver/activer pacemaker. Meme logique, encapsulation conditionnelle differente.

**Entree hosts logstash-seapath**: Une seule tache lineinfile. Identique.

**Configuration libvirt**: Faire utiliser le machine-id par libvirt, redemarrer libvirtd. Puis diverge (CentOS/OracleLinux activent des sockets supplementaires).

**Configuration OVS-vswitchd (Debian et CentOS)**: Creer ovs-vswitchd.service.d, copier le drop-in, copier le service team0. Absent d'OracleLinux.



## Comment corriger: Creer un role de base partage

Creer des roles `roles/common_base` et `roles/common_physical_machine` qui contiennent la logique partagee, avec des variables specifiques aux distributions passees via `include_role` :

```yaml
# Dans roles/debian/tasks/main.yml
- name: Apply common base configuration
  ansible.builtin.include_role:
    name: common_base
  vars:
    vimrc_path: /etc/vim/vimrc
    grub_update_command: update-grub
```

