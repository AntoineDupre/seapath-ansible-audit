### Taches

**[Q&A]**

| Points | Taches | Description |
| ------ | ------ | ----------- |
| [0] | [Amélioration CI](#Amelioration-CI) | Creation de stage dans le CI (Q&A / Molecule Test / Functionnal Test / Report) 
| [1] | [Améliotation linter](#Ameliotation-linter) | Activer les règles manquantes de `ansible-lint` et `yamllint` |
| [1] | [Pre commit](#Pre-commit) | Ajout d'outil permettant de s'affranchir des problemes de Q&A dans les commits. |


**[Refactoring basic]**
| Points | Taches | Description |
| ------ | ------ | ----------- |
| [5] | [Anti-pattern](#anti-pattern) | Supprimer les patterns qui vont à l'encontre de la philosophie de ansible. |
| [2] | [Tâche monolitique](#tache-monolitique) |  Décomposer un maximum les fichiers trop complexe en sous fichiers qui regroupent des éléments indépendants. |
| [3] | [Code dupliqué](#Code-dupliqué) |  Restructurer les roles pour eviter la duplication de code. Ceci concerne principalement les roles d'installation des distributions qui sont globalement identique. |

**[Sécurité]**
| Points | Taches | Description |
| ------ | ------ | ----------- |
| [0] | [Secret proteger](#Secret-protégé) | S'assurer que les secrets n'apparaissent pas dans le log produit par ansible. Utilisation de `no_log: True` |
| [1] | [Ansible Vault](#Ansible-Vault): | Le projet ne comporte pas de secret, mais il est important de sensibiliser les utilisateurs et les rédacteur de playbook aux enjeux de protéger les mots de passes et secret via AnsibleVault |
| [5] | [Porter des scripts](#Audit-Script) | Auditer les scripts déployés par ansible. Le projet contient des scripts qui sont souvent cachés dans la structure d'un role. Les scripts ainsi déployé doivent resté minimaliste et ne pas introduire de logique trop complexe (voir email sécurité snmp sur mailing list seapath|
| [3] | [Utilisateur par service](#Utilisateur-par-service) | S'assurer que les services déployés, particulièrement ceux qui exécute des scripts ou de la logique customisée, s'execute avec des permissions limitées et un utilisateur dédié |
| | [Avertissement Ansible](#Parametres-commandes) | Bien lire les recommandations de la documentation des commandes ansible avant de manipulé les paramètre et changer le comportement par défaut |



**[Production Friendly]**
| Points | Taches | Description |
| ------ | ------ | ----------- |
| [3] | [Documentation des varibables](#Documentation-des-varibables) | Ajouter `default/main.yml` dans tous les roles ansible afin de facilement documenter les variables, exposer l'interface de configuration officielle du role, et données des valeurs par défaut |
| [2] | [Roles de ci](#roles-de-ci) | Déplacer les roles qui ne sont pas en lien avec la configuration de seapath (les roles propres à la configuration du CI) dans un repo annexe. |
| [1] | [Inventaire](#Rennomer-le-dossier-inventaires-en-exemple_inventaire) | Déplacer les inventaires d'exemple dans un dossier `exemple_inventaire` afin de ne pas mélanger les exemples avec les potentielles inventaires ou configurations officielles ou de productions. |
| [5] | [change when](#change-when) | Revoir l'implémentation du pattern `change_when` et s'assurer qu'il correspondent a une logique conditionnelle valide. Il est important que les changements soit correctement reporter a ansible afin de permettre l'utilisation de `--check` en production, afin de vérifier l'état de la configuration | 
| [8] | [Action vers Etats: Idempotency](#Idempotecy-Action-vers-Etat) | Revoir la structure des roles afin d'eviter que les roles ressembles a une série d'action a effectuer. Un role est une description d'état a atteindre sur la machine. L'utilisation de `shell` ou de `commande` doit donc être utiliser avec minutie.  S'assurer que les taches puissent être appliqué deux fois de suite, sans que la deuxieme applications produise de changement. |
| [2] | [Tags](#tags) |  utilisation de tag | Les taches dans des roles peuvent doivent contenir des tags, afin de les identifier. Le minimum est de mettre un tag correspondant au role a toute les taches d'un même role |

**[Test unitaire]**
| Points | Taches | Description |
| ------ | ------ | ----------- |
| [8] | [Test roles](#test-roles) | Les tests actuelles sont des tests fonctionnelles, des tests unitaires indépendant devrait etre ajouté.| 
| [5] | [Test script](#test-script) | Les scripts complexes doivent avoir des tests unitaires afin de garantir leurs logiques.  | 
| [8] | [Test playbook](#test-playbook) | Les playbooks doivent avoir des tests. Les playbooks correspond à l'interface offerte aux utilisateurs, ils doivent etre fiable et testé.| 


### Approche envisagée


L’application de ces recommandations constitue un processus progressif à l’échelle du projet. Les rôles étant relativement indépendants, une stratégie itérative est recommandée.

Cette approche vise à mettre en place les bonnes pratiques décrites ci-dessous en priorité pour :
 - Les rôles critiques (à identifier)
 - Les nouveaux rôles
 - Les rôles faisant l’objet d’un besoin de refactoring
 - Les rôles qui manipulent des scripts

Recommandations lors de l’écriture ou du refactoring d’un rôle:

 - Limiter le périmètre des rôles
 - Organiser les tâches dans plusieurs fichiers (`tasks/main.yml`, `tasks/install.yml`, `tasks/config.yml`, etc.) 
 - Éviter l’utilisation directe de `shell` et `command`
 - Utiliser correctement `changed_when` et `failed_when` afin de: 
    - Garantir une sortie cohérente
    - Éviter les faux positifs
    - Respecter les principes d’idempotence
 - Décrire un état et non une action
 - Ajouter des contrôles d’état lors de l’usage de commandes non natives (exemple `rsync` qui définie les mode d'un fichier)
 - Garantir l’idempotence
 - Ajouter des tests Molecule dans une approche proche du test unitaire. (Si il est difficile d'écrire des tests unitaires, c'est généralement que la structure est mauvaise)
 - Protéger les secrets
   - `no_log: True` pour eviter les fuites de secret dans les log
   - Ansible Vault pour protéger les secrets dans les inventaires
 - Documenter les variables du rôle: 
   - Définir ses variables dans defaults/main.yml
   - Fournir des valeurs par défaut cohérentes


***



Description Taches:
### Amelioration-CI
Voir proposal: https://github.com/seapath/ansible/pull/886

L'idée est d'alléger les tests fonctionnelles de tous les tests qui sont propre au repo ansible. Les machines qui exécutes les tests fonctionnelles sont des ressources limités, et il est préférable de ne pas les surcharger ou les polluer avec des tests qui peuvent etre exécutés dans des environnement plus standard.

La PR propose de séparer les tests Q&A, mais l'idée générale est également d'ajouter une étape de tests unitaires avec molécules (qui ne nécessite pas de machine physique dédiée).
 
<img width="668" height="386" alt="image" src="https://github.com/user-attachments/assets/0a29e8fd-0e7b-49e0-8b04-38dcb6a098da" />

***

### Ameliotation-linter
`ansible-lint` est un linter qui verifie la structure des fichiers yaml et qui vérifie que le projet respecte les conventions, les standards et bonnes pratiques d'ansible.

Des vérifications sont désactivé, et devrait être réactiver afin d'améliorer la qualité global. En terme de qualité, il faut considéré la mentra suivante: 
```
If a rule is not automatically check in your code, it's not your rule.
```

Les règles désactivé sont:
| Regle | Raison | Commentaire |
| ----- | ------ | ----------- |
| unnamed-task | Correspond aux taches de debug | La règle ne devrait pas etre désactivé globalement, mais devrait être explicitement désactiver via `# noqa: ` dans le code roblème réglé dans la PR https://github.com/seapath/ansible/pull/874  |
| yaml | Correspond aux erreurs de plus de 80 caractères par ligne | La règle des 80 characteres doit etre désactivées dans `.yamllint` en ajoutant `line-length: disable` au lieu de désactiver tous le linter yaml.  |
| fqcn-builtins | Version d'ansible trop vieille a cause `ceph-ansible`| La version d'ansible est maintenant 2.16 et permet d'activer cette régle. [(Analyse détaillée)](docs/fqnc.md)
| file permissions | Doit être fait fichier par fichier | Les fichiers sans mode vont utiliser les `umask` par défaut. Il semble que sur toute les distributions de seapath, le umask est le même `0022`. Donc les fichiers ont par défaut le mode `0644` et les répertories `0755`. Réglé dans un PR avec un approche qui utilise `# noqa` plutot que de définir les modes explicitement: https://github.com/seapath/ansible/pull/885 |

Les cas d'usages qui ont amené a désactiver les règle de linter ne justifie pas de désactiver la totalité des vérifications.
Les règles yaml et fqcn peuvent être facilement réglées en utilisant:
```
ansible-lint -c ansible-lint.conf --fix=yaml,fqcn
```
Il est important de prévoir cette opération avec les autres contributeurs, car ces modifications vont impactés la totalité du projet et généré beaucoup de conflits dans les branches et PR existante.





***
### Pre commit

`pre-commit` est un outil exécuté avant chaque commit sur la machine du développeur.  
Il permet d’exécuter automatiquement les linters (par exemple `ansible-lint`, `yamllint`) avant que le code ne soit validé.

Au lieu de découvrir les problèmes uniquement après un push ou lors de l’ouverture d’une Pull Request, les vérifications sont effectuées localement et fournissent un retour immédiat. Cela réduit significativement les échecs de CI causés par :

- des problèmes de formatage mineurs  
- des noms de tâches manquants  
- des erreurs de syntaxe YAML  

Le cycle de feedback est ainsi considérablement raccourci.
Cela garantit que tous les commits respectent les standards définis par le linter.
En conséquence, les pull requests sont plus propres et les revues de code peuvent se concentrer sur la logique, l’architecture et la conception plutôt que sur des corrections de formatage.

Cela garantit également que la version du linter est identique entre l’environnement d’intégration continue (CI) et celui des contributeurs, évitant ainsi les divergences entre les environnements locaux et la CI.

**Coût de mise en place :**
https://docs.ansible.com/projects/lint/configuring/?h=pre#pre-commit-setup

Facile à configurer dans le dépôt du projet : il suffit de se référer au dépôt officiel ansible-lint pour obtenir une configuration standard.


`.pre-commit-config.yaml`
```
---
repos:
  - repo: https://github.com/ansible/ansible-lint
    rev: v26.1.1
    hooks:
      - id: ansible-lint
        # additional_dependencies:
        #   - ansible
```

La principale difficulté est que pre-commit doit être installé sur la machine des développeurs afin de s’exécuter localement avant chaque commit. Les utilisateurs doivent donc accepter ce workflow.
```
pip install pre-commit
pre-commit install
```

***
### Anti pattern
Certaine tache utilise des patterne qui sont contre la philosophie ansible. 
Une analyse approfondie des patterns doit être mené.


Pattern a revoir identifié:

**shell: "{{ item }}"**: [Article détaillé](details/anti_patern.md)

**Wipe approche** Il n'est pas recommandé de supprimer l'intégralité d'un répertoire avant de déployé un fichier. Ceci est fait par exemple [ici](https://github.com/seapath/ansible/blob/6ec11580566ef0296d0c79913af1393252abfb5d/roles/network_basics/tasks/main.yml#L5). Ceci casse le concepte d'idemptency car l'action de suppression sera toujours effectuée.

L'approche recommandé est définir un role qui décrit que seul le fichier que l'on souhaite déployé soit présent.

***
### tache monolitique

[Explication détaillée](details/monolitit.md)

Des fichiers de taches de roles melangent de nombreuses taches indépendantes dans un seul fichier. Cela les rend :

    Difficiles a relire — une PR touchant une section imacte le fichier entier
    Difficiles a reutiliser — on ne peut pas inclure uniquement la section "utilisateur admin" sans tout executer
    Difficiles a tester — pas moyen d'executer des sous-ensembles du role de maniere isolee
    Difficiles a maintenir — trouver une tache specifique necessite de faire defiler des centaines de lignes

Fichiers identifiés:
```
- roles/cephadm/tasks/main.yml
- roles/debian/tasks/main.yml
- roles/centos/tasks/main.yml
- roles/oraclelinux/tasks/main.yml
- roles/yocto/tasks/main.yml
- roles/centos_physical_machine/tasks/main.yml
- roles/debian_physical_machine/tasks/main.yml
- roles/oraclelinux_physical_machine/tasks/main.yml
- roles/debian_hardening/tasks/main.yml
- roles/centos_hardening/tasks/main.yml
```

[Exemple de découpage](https://github.com/AntoineDupre/seapath-ansible/tree/refactor-centos-roles/roles/centos)


***
### Code dupliqué

Du code est dupliqué massivement entre les roles specifiques aux distributions et les roles de machines physiques. Des blocs de taches identiques sont copies-colles entre les variantes `debian`, `centos` et `oraclelinux`. Cela signifie :

- **Les corrections de bugs doivent etre appliquees 3 fois** — une par distribution
- **La divergence est inevitable** — les copies derivent au fil du temps (et c'est deja le cas)
- **La charge de revue est triplee** — les PRs touchant une distribution devraient toucher les trois

[Article détaillé](details/duplicated_code.md)

***
### Secret protégé
Problème réglé: https://github.com/seapath/ansible/pull/880

Ajouter `no_log: true` à toutes les tâches qui manipulent des secrets ou des mots de passe.

Lorsque Ansible exécute une tâche, il enregistre l’ensemble des paramètres de la tâche (entrée et sortie) dans  `stdout` ainsi que dans le fichier `ansible.log`. Si une tâche manipule un mot de passe ou une clé secrète, celui-ci apparaît en clair dans: 
- La sortie terminal de la personne qui exécute le playbook
- Le fichier `ansible.log` sur le disque
- Les logs de CI/CD (GitHub Actions, Jenkins, etc.)
- Le contenu des variables enregistrées si elles sont affichées via `debug`

L’ajout de `no_log: true` indique à Ansible de masquer les entrées et sorties de la tâche dans tous les logs.  ([doc](https://docs.ansible.com/projects/ansible/latest/reference_appendices/logging.html#protecting-sensitive-data-with-no-log))


***
### Ansible Vault
Le projet ne semble pas etre concerné pour le moment. Il est important de s'assurer qu'il n'existe pas de clés ou de secret acessible dans la configuration (génralement dans les inventaires.).

Il est important néanmoins de connaitre `Ansible vault` qui permet de sécuriser les secrets en cas de besoin, ou dans l'écriture d'inventaires clients.

Envisager d'ajouter une section d'information dans le README ou dans la documentation des inventaires, ou dans les inventaires d'exemple.

***
### Audit Script
L’objectif principal d’un dépôt Ansible est de décrire et d’appliquer l’état souhaité des machines à travers des rôles. Les playbooks et rôles doivent idéalement se limiter à la configuration du système et à la gestion de cet état.

Cependant, le dépôt contient également plusieurs scripts déployés sur les machines (shell, Python, etc.). Ces scripts ne participent pas seulement à la configuration : ils embarquent parfois de la logique fonctionnelle exécutée en production. Ils deviennent alors une partie du logiciel réellement déployé, et non plus uniquement de l’automatisation.

Ce mélange entre configuration (Ansible) et logique applicative (scripts) peut compliquer la maintenance, la compréhension et l’évolution du système.
- Garder les scripts courts et simples, limités à des tâches utilitaires ou d’intégration.
- Contrôler précisément leur exécution (services systemd, timers, appels explicites), afin d’éviter des comportements implicites ou difficiles à tracer.
- Documenter clairement leur rôle et leur cycle de vie (quand ils sont exécutés, par qui, avec quels privilèges).
- Pour des composants plus complexes, envisager un packaging dédié (paquet système, projet séparé, dépôt distinct), plutôt que de les maintenir directement dans les rôles Ansible.

L’objectif est de maintenir une séparation claire entre l’automatisation de la configuration et le code exécuté en production, afin de conserver des rôles Ansible lisibles, maintenables et prévisibles.

**Python script:**
```
- roles/backup_restore/files/scripts/backup_du.py
- roles/backup_restore/files/scripts/get_metadata.py
- roles/backup_restore/files/scripts/remove_disk_xml.py
- roles/ci_yocto/get_system_info/files/get_system_info.py
- roles/ci_yocto/run_tests/files/run_cyclictest.py
- roles/configure_nic_irq_affinity/files/setup_nic_irq_affinity.py
- roles/debian_tests/cukinia-tests/includes/ioperm.py
- roles/debian_tests/cukinia-tests/includes/prctl.py
- roles/debian_tests/cukinia-tests/includes/ptrace.py
- roles/ptp_status_vsock/files/ptp_vsock.py
- roles/snmp/files/snmp_getdata.py
- roles/vmmgrapi/files/wsgi.py
```


**Bash script:**
```
- roles/backup_restore/files/scripts/backup-restore.sh
- roles/backup_restore/files/scripts/backup_full.sh
- roles/backup_restore/files/scripts/backup_inc.sh
- roles/backup_restore/files/scripts/edit_metadata.sh
- roles/backup_restore/files/scripts/edit_vmxml.sh
- roles/backup_restore/files/scripts/restore_vm.sh
- roles/ci_yocto/reboot_on_usb_drive/files/configure_boot_next_usb_drive.sh
- roles/debian_hardening/files/mktmpdir.sh
- roles/debian_hardening/files/terminal_idle.sh
- roles/deploy_cephfs/files/wait-for-mds.sh
- roles/ptp_status_vsock/files/ptpstatus/ptpstatus.sh
- roles/snmp/files/scripts/virt-df.sh
```

**Complexité des scripts**

Les scripts complexes doivent avoir des tests unitaires afin de garantir le bon fonctionnement. (cf 

| # | File | Lines | Why it needs tests |                                                                                                                    
  |---|------|-------|--------------------|
  | 1 | `snmp/files/snmp_getdata.py` | 348 | Multi-system health aggregation, embedded shell commands, XML/JSON parsing, disk-replace state machine |          
  | 2 | `ptp_status_vsock/files/ptpstatus/ptpstatus.sh` | 631 | IEC 61850 SmpSync state machine, hex arithmetic, `pmc` output parsing |                        
  | 3 | `backup_restore/files/scripts/backup-restore.sh` | 212 | 14 functions, multi-level TUI menus, config read/write |                                      
  | 4 | `snmp/files/scripts/virt-df.sh` | 80 | RBD map/unmap, LVM diff via associative arrays, multi-fstype mount logic |
  | 5 | `backup_restore/files/scripts/restore_vm.sh` | 83 | Multi-step restore pipeline, incremental diff application, metadata restore |
  | 6 | `backup_restore/files/scripts/edit_metadata.sh` | 80 | Two-level TUI, nameref arrays, editor integration, file change detection |
  | 7 | `backup_restore/files/scripts/backup_full.sh` | 56 | RBD snapshot loop, nested metadata loop, include/exclude regex filtering |
  | 8 | `backup_restore/files/scripts/backup_inc.sh` | 55 | Incremental RBD diff export, latest snapshot detection |
  | 9 | `configure_nic_irq_affinity/files/setup_nic_irq_affinity.py` | 73 | `/proc/irq` traversal, argparse, NIC/CPU count validation |
  | 10 | `backup_restore/files/scripts/backup_du.py` | 75 | `rbd du` output parsing, unit conversion, dict accumulation |
  | 11 | `ptp_status_vsock/files/ptp_vsock.py` | 50 | VSOCK threaded server, message routing, retry loop |

**Risk: command injection**

Voir mailing list SEAPATH

- Supprimer les pattern `supbprocess.checkout_output(cmd, shell=True ...)`
- Faire attention avec `source` qui utilise des fichiers qui peuvent etre édité (vigilence `backup-restore.sh`)
- Faire attention avec `$remote_shell`  (vigilence `backup_full.sh`, `backup_inc.sh`)



***
### Utilisateur par service
Lors de la création d’un service systemd, il est recommandé d’appliquer le principe du moindre privilège. Le service ne doit disposer que des droits strictement nécessaires.

**1. Utilisateur dynamique (recommandé par défaut)**
Si le service n’a pas besoin d’un UID stable ni d’accéder à des ressources partagées, utiliser:
```
DynamicUser=yes
```
Cela crée un utilisateur éphémère géré automatiquement par systemd.
Pour les fichiers nécessaires au service: 
 - `RuntimeDirectory=`: crée un répertoire temporaire dans`/run` (données volatiles).
 - `StateDirectory=`: crée un répertoire persistant dans `/var/lib`.

Ces répertoires sont automatiquement créés avec les permissions adaptées.

**2. Utilisateur dédié (si permissions ou capacités spécifiques)**
Si le service doit accéder à des fichiers existants, partager des ressources ou utiliser des permissions, il est préférable de créer un utilisateur système dédié (via Ansible par exemple) et de définir explicitement:
```
# Service file
User=service_user
Group=service_user
```
Les permissions des fichiers et répertoires doivent alors être gérées explicitement dans l’automatisation.

Ansible doit gérer les permissions et la creation de l'utilisateur:
```
- name: Create ptp dedicated user
  ansible.builtin.user:
    name: ptp
    system: true
    home: /var/lib/ptp
    shell: /usr/sbin/nologin
    create_home: false

- name: Ensure ptp directory owned by ptp
  ansible.builtin.file:
    path: /var/lib/ptp
    state: directory
    owner: ptp
    group: ptp
    mode: "0750"
```

** Exécution en root (à éviter) **
Un service ne doit être exécuté en root que s’il n’existe pas d’alternative (accès matériel, gestion réseau privilégiée, opérations système critiques).
Dans la mesure du possible, préférer un utilisateur dédié avec des permissions systemd ciblées.


#### Etat des service:

Service avec utilisateurs définis (5/18) 
| File | Setting |                                                                                                                                           
  |------|---------|                                        
  | `centos_physical_machine/templates/chrony-wait.service.j2` | `DynamicUser=yes` |                                                                           
  | `debian_physical_machine/templates/chrony-wait.service.j2` | `DynamicUser=yes` |
  | `oraclelinux_physical_machine/templates/chrony-wait.service.j2` | `DynamicUser=yes` |
  | `ptp_status_vsock/files/ptp_vsock.service` | `DynamicUser=yes` |
  | `vmmgrapi/files/gunicorn.service` | `User=root` (with `# DynamicUser=yes` commented out) |

Service avec utilisateur root (13/18)
  | File | Notes |
  |------|-------|
  | `centos_physical_machine/files/team0_x@.service` | Runs as root implicitly |
  | `ci_yocto/synchronise_vms/templates/phc2sys.service.j2` | Runs as root implicitly |
  | `configure_nic_irq_affinity/files/setup_nic_irq_affinity.service` | Writes to `/proc/irq` — needs root |
  | `debian_grub_bootcount/files/system_check.service` | Runs as root implicitly |
  | `debian_hardening/files/random-root-passwd.service` | Runs as root implicitly |
  | `debian_physical_machine/files/team0_x@.service` | Runs as root implicitly |
  | `deploy_cephfs/files/ceph-mds-ready@seapathcephfs.service` | Runs as root implicitly |
  | `deploy_python3_setup_ovs/files/seapath-config_ovs.service` | Runs as root implicitly |
  | `network_clusternetwork/files/hsr.service` | Runs as root implicitly |
  | `network_networkdwait/templates/systemd-networkd-wait-online.service.j2` | Runs as root implicitly |
  | `ptp_status_vsock/files/ptpstatus/ptpstatus.service` | Runs as root implicitly |
  | `timemaster/templates/timemaster.service.j2` | Runs as root implicitly |
  | `yocto/sriov/templates/sriov-configure.service.j2` | Runs as root implicitly |

13 services s’exécutent tous en root par défaut.
Certains ont légitimement besoin des privilèges root ( IRQ, configuration réseau, PTP, Ceph MDS). Il serait préférable de donner les permissions avec un utilisateur dédié.
D’autres, comme `ptpstatus.service` ou `seapath-config_ovs.service`, pourraient potentiellement être renforcés en utilisant DynamicUser=yes ou un utilisateur de service dédié.
***

### Parametres commandes
Les commandes ansible peuvent être paramétrés. Il est recommandé de bien lire la documentation lorsque des paramètres sont changés de leur valeur par défaut.  Un role ne connait pas forcément sont contextes d'exécution (nombre d'hote, stratégie de concurence, execution serie/parrallele ...), les paramètres par défaut sont pensé pour s'exécutrer de manière fiable. 

Exemple. La command `fetch` de ansible permet de récupérer des données depuis différent hotes. Par defaut les variables provenant de différent hotes avec de même nom sont protégés par le fait que ansible ajoute une structure de fichier `/path/to/file `

Ceci peut etre outrepassé via le parametre [flat](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/fetch_module.html#parameter-flat) La documentation spécifie:
```
This can be useful if working with a single host, or if retrieving files that are uniquely named per host.

If using multiple hosts with the same filename, the file will be overwritten for each host.
```

Cette PR illustre les problèmes de concurence qui peuvent se produire lorsque cette option n'est pas par défaut: https://github.com/seapath/ansible/pull/876

***

### Documentation des varibables
Les utilisateurs doivent écrire ou adapter leurs propres inventaires et définir les variables nécessaires à l’exécution des rôles. Dans ce contexte, il est important que les variables attendues par chaque rôle soient clairement identifiées et documentées afin de faciliter leur utilisation et leur configuration.

#### Ajout de fichiers `defaults/main.yml` dans les rôles Ansible

Peu de rôles Ansible disposent de fichier `defaults/main.yml`. Ajouter ce fichier ce fichier dans les roles permet de centraliser l'ensemble de paramètre de configuration possible. 
Les variables sont bien présentent dans les README, mais le fichier `defaults/main.yml` permet de :
- documenter clairement les variables configurables du rôle
- définir des valeurs par défaut garantissant un fonctionnement minimal
- faciliter la réutilisation et la surcharge des variables par les playbooks ou les inventaires.
- expose la configuration du rôle (permet de s'intégrer avec des outils comme ansible galaxy)

Sa présence améliore la lisibilité, la maintenabilité et la standardisation des rôles dans le projet.

** Types de variables **
`defaults/main.yml` définit les variables configurables d’un rôle avec la priorité la plus faible dans Ansible.

1 - Variable optionnelle:
Variable configurable mais non obligatoire.
 - possède une valeur par défaut cohérente
 - ne nécessite pas de validation dans le role

```yaml
# defaults/main.yml
app_port: 8080
app_user: myapp
```

2 - Variable obligatoire (sans valeur par défaut possible)
Le rôle ne peut pas fonctionner sans cette variable.
- déclarée dans `defaults/main.yml`
- valeur explicite `null`
- validation avec `assert`

```yaml
# defaults/main.yml
cluster: null
```
```yaml
# tasks/validate.yml
- assert:
    that:
      - cluster is not none
    fail_msg: "cluster is not defined"
```

3 - Variable optionnelle activant une fonctionnalité
La variable active une fonctionnalité seulement si elle est définie.
- valeur par défaut `null`
- condition `when`
```yaml
# defaults/main.yml
monitoring_endpoint: null
```
```yaml
when: monitoring_endpoint is not none
```

Mauvaises pratiques:
- Variable  non documentée: Variable utilisée dans les tâches mais absente de defaults.
- Variable vide: Ambigu et difficile à tester.
```yaml
cluster:
```
Roles sans `defaults/main.yml`:

`add_libvirtadmin_user`, `backup_restore`, `centos`, `centos_hypervisor`, `centos_physical_machine`, `ceph_expansion_lv`, `ceph_expansion_vg`, `ceph_prepare_installation`, `ci_centos`, `ci_cleanup_varlog`, `ci_reinstalliso`, `ci_restore_snapshot`, `ci_restoredd`, `ci_yocto`, `configure_libvirt_rdb_secret`, `configure_nic_irq_affinity`, `debian`, `debian_hardening`, `debian_hardening_physical_machine`, `debian_hypervisor`, `debian_physical_machine`, `debian_tests`, `deploy_cockpit_plugins`, `deploy_cukinia`, `deploy_python3_setup_ovs`, `deploy_vm_manager`, `detect_seapath_distro`, `hardware_customization_welotec`, `iptables`, `network_basics`, `network_buildhosts`, `network_clusternetwork`, `network_guestsinterfaces`, `network_netplan`, `network_networkdwait`, `network_resolved`, `network_sriovpool`, `oraclelinux`, `oraclelinux_physical_machine`, `oraclelinux_tests`, `ptp_status_vsock`, `snmp`, `yocto`


#### Ajouter des modèles `group_vars/`

Créer un répertoire group_vars/ contenant des fichiers .example pour documenter les variables attendues par groupe. Ces fichiers serviraient de modèles de configuration pour les utilisateurs.

```
group_vars/
├── all.yml.example
├── cluster_machines.yml.example
├── hypervisors.yml.example
├── osds.yml.example
├── standalone_machine.yml.example
└── VMs.yml.example
```
`group_vars/all.yml.example`:

```yaml
# ===========================
# SEAPATH Global Variables
# ===========================

# --- Admin User ---
# admin_user: "admin"          # Required. Username for the admin account
# admin_passwd: ""             # Optional. Hashed password (use `mkpasswd --method=sha-512`)
# admin_ssh_keys: []           # Optional. List of SSH public keys

# --- Network ---
# gateway_addr: "192.168.1.1"  # Required. Default gateway IP
# dns_servers:                  # Optional. List of DNS servers
#   - "8.8.8.8"
# subnet: 24                   # Optional. Default: 24
# ntp_servers:                  # Optional. List of NTP servers
#   - "pool.ntp.org"
```

`group_vars/cluster_machines.yml.example`:

```yaml
# ===========================
# Cluster Machine Variables
# ===========================

# --- Cluster Networking (Required) ---
# cluster_network: "192.168.55.0/24"     # Ceph cluster network CIDR
# public_network: "{{ cluster_network }}" # Usually same as cluster_network
# team0_0: "eth1"                         # First bonding interface
# team0_1: "eth2"                         # Second bonding interface

# --- Per-Host Variables (set in inventory, not here) ---
# cluster_ip_addr        - This host's cluster IP
# cluster_next_ip_addr   - Next node's cluster IP (for ring topology)
# cluster_previous_ip_addr - Previous node's cluster IP
# monitor_address        - Usually "{{ cluster_ip_addr }}"
```

***
### roles de ci
Voir PR https://github.com/seapath/ansible/pull/886

***
### Rennomer le dossier inventaires en exemple_inventaire 
Le projet contient aujourd’hui un répertoire `inventories/` qui regroupe principalement des inventaires d’exemples.

Dans la pratique, `inventories/` va être le répertoire utilisés par les clients pour décrires leurs configuration.  Ils vont surement fork/clone le dépôt et construisent leur propre configuration dessus, où avoir leur propre dossier d’inventaire qu’ils souhaitent intégrer. Les inventaires d'exemple ne doivent pas cohabiter avec des inventaires de production ou provoqué des collision

La recommandation est donc de restructurer l’arborescence en séparant clairement :
- un espace destiné aux inventaires “utilisateur / production” (`inventories/`), et pouvant contenir des definitions de variables propres a des fournisseurs
- un espace explicitement dédié aux exemples (`examples_inventories/`)

Certain exemples peuvent être améliorer en introduisant une structure standardisée avec `group_vars/` et `host_vars/`.

***
### change when

**104 taches** dans le depot utilisent `changed_when: true`, ce qui marque inconditionnellement chaque execution comme "modifiee". Cela casse le modele d'idempotence d'Ansible :

- **Impossible de savoir si une execution de playbook a reellement modifie quelque chose** — chaque execution rapporte des changements
- **Les handlers se declenchent toujours** — les taches qui font `notify` a un handler le declenchent a chaque execution, meme quand rien n'a change
- **La piste d'audit est inutile** — le mode `--check` et les rapports de changements deviennent sans signification

L'approche correcte est d'inspecter la sortie de la commande et de ne rapporter un changement que lorsque quelque chose a reellement change.

Dans l'ensemble, il est préférable d'utiliser tant que possible des instructions native d'ansible.

#### Comment corriger

##### Pattern 1 : Les commandes en lecture seule doivent utiliser `changed_when: false`

Les commandes qui ne font que **consulter** l'etat ne modifient jamais rien :

```yaml
# AVANT
- name: Get root user's home directory
  shell:
    cmd: set -o pipefail && getent passwd root | cut -d ':' -f6
    executable: /bin/bash
  register: result
  changed_when: true       # FAUX — c'est en lecture seule

# APRES
- name: Get root user's home directory
  shell:
    cmd: set -o pipefail && getent passwd root | cut -d ':' -f6
    executable: /bin/bash
  register: result
  changed_when: false      # Cette commande ne modifie jamais rien
```

##### Pattern 2 : Les commandes avec une sortie detectable doivent utiliser une logique conditionnelle

De nombreuses commandes produisent une sortie qui indique si un changement a eu lieu :

```yaml
# AVANT
- name: Install cephadm
  command: /tmp/cephadm install
  changed_when: true

# APRES
- name: Install cephadm
  command: /tmp/cephadm install
  register: cephadm_install_result
  changed_when: "'already' not in cephadm_install_result.stdout"
```

```yaml
# AVANT
- name: Create RBD pool if it doesn't exist
  command: ceph osd pool create rbd
  changed_when: true

# APRES
- name: Create RBD pool if it doesn't exist
  command: ceph osd pool create rbd
  register: pool_create_result
  changed_when: "'already exists' not in pool_create_result.stderr"
```

##### Pattern 3 : Les handlers doivent toujours rapporter un changement

Les handlers sont conçus pour ne s'executer que lorsqu'ils sont notifies, donc `changed_when: true` est acceptable ici puisqu'ils ne s'executent que lorsqu'une tache declencheuse a reellement change :

```yaml
# Ceux-ci sont OK — les handlers ne s'executent que lorsqu'ils sont notifies
- name: Update-grub
  command: update-grub
  changed_when: true    # Acceptable dans un handler
```

##### Pattern 4 : Les operations ponctuelles gardees par `when:` sont acceptables

Si une tache a deja une condition `when:` qui empeche la re-execution, `changed_when: true` est moins nuisible mais reste imprecis :

```yaml
# Moins critique — le garde when: empeche les re-executions
- name: Bootstrap Ceph cluster
  command: cephadm bootstrap ...
  when: cephadm_do_bootstrap
  changed_when: true    # Acceptable si when: empeche la re-execution
```

`changed_when: true` peut mentir au moteur de convergence dans ce cass si la condition est vrai mais que la command n’a rien modifié ou a échoué., Les opérations ponctuelles peuvent être acceptées si elles sont gérées de manière idempotente (garde + preuve d’état) ; `changed_when: true` reste un dernier recours et doit être justifié.

[Liste détaillés des fichiers concernés](details/changed_when_list.md)

***
### Idempotecy Action vers Etat
[Action continue a avoir] Ceci n'est pas réellement une tache, mais une recommandation à avoir en tete lorsque l'on ecrit un role, ou lorsque des modificaitions sont apportés.

Dans Ansible, on ne décrit pas ce que l’on veut faire,on décrit l’état que l’on souhaite obtenir.
Un playbook doit exprimer ce que le système doit être, pas ce que l’on veut faire.

Cette approche est au coeur du principe d’idempotence : relancer un playbook ne doit pas provoquer de modifications si l’état cible est déjà atteint

Dans ce contexte, l’utilisation de commandes génériques (`command`, `shell`, etc.) doit être accompagnée de vérifications explicites de l’état du système, en particulier lorsque l’on ne dispose pas de module Ansible natif. Ces tâches de contrôle permettent de:
- déterminer si une action est réellement nécessaire
- éviter les exécutions répétées inutiles 
- améliorer la précision du statut changed
- rendre le rôle plus fiable et prévisible

Un exemple de cette approche peut être observé dans la PR suivante: https://github.com/seapath/ansible/pull/877

L’objectif est d’ajouter des tâches de vérification d’état avant ou après l’exécution d’une commande afin de déterminer si l’état cible est déjà atteint ou s’il vient effectivement d’être modifié.


***
### Tags
Il est recommandé d’utiliser des tags dans les rôles Ansible afin d’améliorer la flexibilité d’exécution des playbooks et de faciliter les opérations en environnement de production.

Chaque rôle devrait au minimum être associé à un tag portant son nom. Cela permet de cibler ou d’exclure facilement un rôle lors de l’exécution d’un playbook, par exemple :
```bash
ansible-playbook site.yml --tags logging
ansible-playbook site.yml --skip-tags network
```
Cette pratique rend les playbooks plus modulaires et opérables, notamment pour :
- exécuter uniquement une partie de la configuration
- isoler un rôle lors d’un diagnostic ou d’un correctif (fonctionne avec --check)
- éviter certaines opérations dans un contexte particulier

Il est également utile d’introduire des tags fonctionnels pour qualifier certains comportements spécifiques, par exemple :
- `not_idempotent` : pour identifier des tâches ou rôles qui ne sont pas strictement idempotents
- `dangerous` : pour les opérations sensibles ou potentiellement destructives
- `raw_cmd` : pour les executions de commands ou de shell.

Ces tags permettent aux opérateurs de désactiver facilement certaines catégories d’actions, par exemple :
```
ansible-playbook site.yml --skip-tags raw_cmd --check
```




***
### test roles
Les roles peuvent être tester indépendament des tests de CI actuelle.

Les tests de CI sont des tests fonctionnel, les roles doivent être testé unitairement.

Un proposal est ouvert sur le upstream: https://github.com/seapath/ansible/pull/875

Ce qu'il faut retenir:
 - Molecule permet l'execution de role dans des environements simple
 - Molecule permet de configurer un environement de test pour le role, en proposant diverse scénario
 - Molecule permet de tester le resultat des roles via des playbook de tests.
 - Les tests unitaires sont définis au niveau des roles
 - L'ajout d'un [orchestrateur](https://github.com/AntoineDupre/seapath-ansible/blob/proposal-molecule-test/tox.ini) comme `tox` permet d'exécuter tout les tests molécules depuis la racine du projet
 - `Tox` permet egalement de tester les roles avec différentes version d'ansible.


Des exemples de tests sont disponible dans des branches de mon fork (pourront etre merger dans le upstream une fois le proposal accepté).

1 - [Exemple Basic](https://github.com/AntoineDupre/seapath-ansible/tree/proposal-molecule-test/roles/network_basics)

Test de `roles/network_basics` en illustrant la capacité de créé des scénarios et de cpnfigurer les variables du roles.

2 -  [Test avec cluster (plusieurs machines)](https://github.com/AntoineDupre/seapath-ansible/tree/molecule-libvirtadmin/roles/add_libvirtadmin_user)

Exemple de test plus complexe, utilisant plusieurs instance de test afin de vérifier la copie de clé entre les différentes instances.

3 - [Test systemd + environement ressources partagés + custom image](https://github.com/AntoineDupre/seapath-ansible/tree/refactor-centos-roles/roles/centos)

Un exemple de restructuration d'un roles complexes. Lorsqu'un role est difficile a tester, il est souvent important de le restructurer en tache plus unitaire. Ici, on décompose `roles/centos/tasks/main.yml` en un ensemble de sous fichier indépendant. Chaque fichier peut ainsi avoir son propre environement de test. 

Le test montre également comment il est possible entre les scénarios de partager des environements ou des fichiers de configuration en commun.

Cette exemple montre egalement comment il est possible d'ajouter ces propres images docker dans le processus de test (pas forcément necessaire ici).

De plus, cette exemple permet d'illustrer les étapes nécessaire pour tester les services en s'assurant que `systemd` soit bien présent.

 
 4 - [Mock dans molecule](https://github.com/AntoineDupre/seapath-ansible/tree/mock-molecule-test/roles/network_sriovpool)

Un exemple d'environement ou l'utilitaire de gestion de VM ne peut être simplement installé. L'exemple montre ainsi comment mock la commande et garder un historique des appels qui peut ensuite etre utilsé pour s'assurer de la bonne exécution. Il serait egalement possible de configurer via le mock le retour des commandes, ou générer des side_effects.


 5 -  [Verifier Testinfra](https://github.com/AntoineDupre/seapath-ansible/tree/testinfra--backup-restore/roles/backup_restore)

Un exemple de test molecule qui utilise un `verifier` différent, testinfra. Cela permet de tester le résultat du role ansible via des tests python standard, compatible avec pytest. Il est ainsi possible de faire des tests avancé, et d'utiliser des outils (coverage, metrics, framework ...) de tests avancé.

Il est nottament possible d'utiliser des `fixtures` ou des tests paramétrisés. 
```python
def _scripts():
    scripts_dir = Path(os.environ["SCRIPTS_SRC_DIR"])
    return sorted(
        p.name
        for p in scripts_dir.iterdir()
        if p.is_file() and not p.name.startswith(".")
    )


@pytest.fixture(params=_scripts(), ids=str)
def deployed_script_name(request):
    return request.param

def test_scripts_are_present(host, deployed_script_name):
    f = host.file(str(DEST_DIR / deployed_script_name))
    assert f.exists
    assert f.is_file
    assert f.user == "root"
    assert f.group == "root"
```
Ici est ainsi possible de récupérer dynamiquement les fichiers de scripts qui doivent être déployé et de s'assurer qu'ils ont bien été déployé. Le tests sera exécuté autant de fois qu'il y a de script. Si deux fixtures sont utilisées, toutes les combinaisons possibles seront testées.



#### Note
claude ai en mode agent est assez compétent pour générer l'environement de test molecule, a partir du moment où il peut s'inspirer d'un exemple. En lui donnant l'exemple fait dans `network_basics`, il est capable de génrer les fichiers `prepare.yml`, `converge.yml` et `molecule.yml` de manière assez pertinente. De la meme maniere pour génrer des tests qui necessite un mock ou systemd, `claude` a besoin d'un exemple de test.

Les tests réaliser dans `verifier.yml` sont a revoir car pas toujours pertinent. `claude` est donc très utile pour générer l'environement de test a partir d'un exemple, installer les pacquets, génrer des scénarios de tests (des iptables par exemple).


Une tentative de [commande claude](claude/molecule.md) est disponible

***
### test script

Le projet déploie beaucoup de script. Ceci est de la logique qui va être déployé en production. 

Molecule permet egalement de tester ces scripts, en déployant en environement de tests, et en ajoutant dans `converge.yml` ou `verify.yml` des tests qui invoquent les scripts, ou s'assure du fonctionnement des scripts si ils sont exécuté par des services.

Il est judicieux, en plus des tests molecules qui test le déploiment, d'ajouter des scénarios de tests qui tests également la logique des tests.

Voici la liste des scripts du projet qui idéalement devrait etre testé.

```
- library/cluster_vm.py
- roles/backup_restore/files/scripts/backup_du.py
- roles/backup_restore/files/scripts/get_metadata.py
- roles/backup_restore/files/scripts/remove_disk_xml.py
- roles/ci_yocto/get_system_info/files/get_system_info.py
- roles/ci_yocto/run_tests/files/run_cyclictest.py
- roles/configure_nic_irq_affinity/files/setup_nic_irq_affinity.py
- roles/debian_tests/cukinia-tests/includes/ioperm.py
- roles/debian_tests/cukinia-tests/includes/prctl.py
- roles/debian_tests/cukinia-tests/includes/ptrace.py
- roles/ptp_status_vsock/files/ptp_vsock.py
- roles/snmp/files/snmp_getdata.py
- roles/vmmgrapi/files/wsgi.py
- scripts/get_osd.py
```
```
- roles/backup_restore/files/scripts/backup-restore.sh
- roles/backup_restore/files/scripts/backup_full.sh
- roles/backup_restore/files/scripts/backup_inc.sh
- roles/backup_restore/files/scripts/edit_metadata.sh
- roles/backup_restore/files/scripts/edit_vmxml.sh
- roles/backup_restore/files/scripts/restore_vm.sh
- roles/ci_yocto/reboot_on_usb_drive/files/configure_boot_next_usb_drive.sh
- roles/debian_hardening/files/mktmpdir.sh
- roles/debian_hardening/files/terminal_idle.sh
- roles/deploy_cephfs/files/wait-for-mds.sh
- roles/ptp_status_vsock/files/ptpstatus/ptpstatus.sh
- roles/snmp/files/scripts/virt-df.sh
```


Un exemple de test pour `roles/backup_restore/files/scripts/remove_disk_xml.py` est disponible dans [cette branche](https://github.com/AntoineDupre/seapath-ansible/tree/molecule-test-script/roles/backup_restore)
```
roles/backup_restore/molecule/default/molecule.yml: Scénario basé sur Docker avec l’idempotence ignorée (l’exécution de scripts n’est pas idempotente par nature)

roles/backup_restore/molecule/default/prepare.yml:  Installe `python3-lxml`, copie le script dans le conteneur et crée deux fixtures XML de test :
- Une avec 2 éléments `<disk>` plus d’autres périphériques
- Une sans éléments `<disk>`

roles/backup_restore/molecule/default/converge.yml: Exécute `remove_disk_xml.py` sur les deux entrées de test (aucun rôle Ansible impliqué, exécution pure du script)

roles/backup_restore/molecule/default/verify.yml:  Vérifie :
- Le fichier de sortie existe
- Tous les éléments `<disk>` ont été supprimés
- Les éléments non liés aux disques (`<interface>`, `<console>`) sont conservés
- La structure du domaine (`<name>`, `<memory>`, `<devices>`) est préservée
- Un XML sans disques est transmis sans modification

### Différence clé avec les tests de rôle

Le fichier `converge.yml` utilise `ansible.builtin.command` pour exécuter directement le script Python au lieu d’appliquer un rôle Ansible. Cela démontre que Molecule peut servir de **framework de tests unitaires pour scripts** le conteneur fournit un environnement propre et reproductible pour tester le comportement du script avec des entrées contrôlées et des sorties vérifiées.
```
***
### test playbook

Molecule peut aussi servir a tester les playbooks. Les playbooks sont les interfaces mis a dispositions des utilisateurs. 

Il est important de s'assurer que les playbooks fonctionnent donc correctement et de tester la logique de configuration et la variable utilisateurs.

Les tests de playbook sont plus compliqué a mettre en places.

Ils permettent de tester les roles en profondeurs, en testant les connexions et les variables inter-roles


Exemple de test molecule sur pour playbook [playbooks/cluster-setup-user](https://github.com/AntoineDupre/seapath-ansible/tree/molecule-playbook/playbooks/molecule/cluster_setup_users)

Note : La vérification est difficile dans les playbooks. Ici, les playbooks invoquent principalement des rôles. `verify.yml` vérifie finalement les tâches de chaque rôle, ce qui relève normalement du périmètre des tests Molecule de chaque rôle. La vérification d’un tel playbook ne doit pas reproduire les mêmes vérifications que celles effectuées dans les rôles. Les tests garantissent plutôt la bonne exécution du playbook et la compatibilité des rôles entre eux. Le fichier `verify.yml` pourrait être vide, ou seulement tester des éléments propres à la configuration. La valeur ajoutée d’un tel test, pour un playbook contenant peu de logique, reste la garantie de sa bonne exécution.
```
                                                                                                                                                             
  ### playbooks/molecule/cluster_setup_users/molecule.yml

- 2 instances Docker (node1, node2) présentes à la fois dans les groupes `cluster_machines` et `hypervisors`:  simulant un cluster à 2 nœuds
- Idempotence ignorée (le rôle utilise des tâches `shell` et `command` qui rapportent toujours un changement)

### playbooks/molecule/cluster_setup_users/prepare.yml

- Installe `openssh-server` / `openssh-client`
- Crée le groupe `libvirt` (prérequis pour la création des utilisateurs)
- Génère les clés SSH hôte `ed25519` (nécessaires pour l’échange des `known_hosts`)

### playbooks/molecule/cluster_setup_users/converge.yml

- Importe le playbook réel via `import_playbook: ../../cluster_setup_users.yaml`:  c’est la différence clé avec les tests de rôle.  
  Nous testons le playbook réel tel quel.

### playbooks/molecule/cluster_setup_users/verify.yml

Vérifie sur les deux nœuds :

- L’utilisateur `libvirtadmin` existe avec le shell `/bin/bash` et le groupe principal `libvirt`
- Le compte est déverrouillé dans `/etc/shadow`
- La paire de clés SSH de root a été générée
- `authorized_keys` contient les clés du nœud pair
- `known_hosts` contient les entrées du nœud pair

---

### Points clés démontrés

- **Test de playbook vs test de rôle** : `converge.yml` utilise `import_playbook` pour exécuter le playbook réel sans modification
- **Multi-nœuds** : 2 conteneurs avec des groupes d’inventaire simulent un cluster, ce qui permet de tester la logique d’échange de clés SSH
- **Sans infrastructure lourde** : uniquement des conteneurs Docker avec `openssh` et un groupe `libvirt` pas de Ceph, pas de démon libvirt
```
***
