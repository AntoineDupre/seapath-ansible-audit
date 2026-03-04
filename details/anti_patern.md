
4 taches dans le depot executent des commandes arbitraires depuis des variables via le module `shell` ou `command` dans une boucle. Ce pattern :

- **Est un risque de securite** — tout utilisateur qui controle la variable d'inventaire peut executer du code arbitraire sur les hotes cibles en tant que root
- **Est impossible a auditer** — les commandes reelles ne sont pas visibles dans le playbook, seulement a l'execution
- **Contourne ansible-lint** — toutes les instances utilisent `tags: [skip_ansible_lint]` pour supprimer les avertissements
- **Est duplique** — 3 des 4 instances sont du code identique dans differents playbooks

## Toutes les instances

### Instance 1 — Playbook prerequis Debian

**Fichier :** `playbooks/seapath_setup_prerequisdebian.yaml:95`

```yaml
- name: Run extra commands after upload
  shell: "{{ item }}"
  tags:
    - skip_ansible_lint
  loop: "{{ commands_to_run_after_upload }}"
  when: commands_to_run_after_upload is defined
```

La variable `commands_to_run_after_upload` est une liste de chaines de commandes shell arbitraires definies dans l'inventaire.

### Instance 2 — Playbook prerequis OracleLinux

**Fichier :** `playbooks/seapath_setup_prerequisoraclelinux.yaml:87`

```yaml
- name: Run extra commands after upload
  shell: "{{ item }}"
  tags:
    - skip_ansible_lint
  loop: "{{ commands_to_run_after_upload }}"
  when: commands_to_run_after_upload is defined
```

Identique a la version Debian.

### Instance 3 — Playbook prerequis Yocto

**Fichier :** `playbooks/seapath_setup_prerequisyocto.yaml:59`

```yaml
- name: Run extra commands after upload
  shell: "{{ item }}"
  tags:
    - skip_ansible_lint
  loop: "{{ commands_to_run_after_upload }}"
  when: commands_to_run_after_upload is defined
```

Identique aux versions Debian et OracleLinux.

### Instance 4 — Role d'expansion Ceph

**Fichier :** `roles/ceph_expansion_lv/tasks/main.yml:112`

```yaml
- name: Resize ceph-osd
  command: "{{ item }}"
  with_items:
    - "/usr/bin/ceph-bluestore-tool bluefs-bdev-expand --path {{ ceph_expansion_lv_ceph_osd_number.files[0].path }}"
    - "/usr/bin/ceph-bluestore-tool bluefs-bdev-expand --path {{ ceph_expansion_lv_ceph_osd_number.files[0].path }}"
  changed_when: true
```

Cette instance presente egalement :
- L'utilisation de `with_items:` obsolete au lieu de `loop:`
- L'execution de la meme commande deux fois (probablement une erreur de copier-coller)
- L'utilisation de `changed_when: true` 

## Comment corriger



### Pour les instances 1-3 : 

Voir si l'exécution de commande arbitraire à travers un variable est vraiment utile. 
La recommendation est de laisser l'utilisateur ecrire sont playbook qui exécute ce qu'il souhaite customiser plutot que d'implémenter les pattern d'exécution arbitraire de code. Typiquement, il peut y avoir des secrets dans les commandes, qui ne seront pas protégé (pas de `no_log: True` et on ne peut pas desactiver les log pour toutes les commandes).


Si malgré tout, cette fonctionnalité doit persister:
 - Extraire dans un script :mettre toutes les commandes post-upload dans un seul script deploye sur l'hote
 - Dedupliquer le code partage: Puisque les 3 playbooks prerequis ont la tache identique, l'extraire dans un fichier de taches partage :

```yaml
- name: Deploy post-upload script
  ansible.builtin.template:
    src: post_upload.sh.j2
    dest: /tmp/post_upload.sh
    mode: '0755'
  when: commands_to_run_after_upload is defined

- name: Run post-upload script
  ansible.builtin.command:
    cmd: /tmp/post_upload.sh
  when: commands_to_run_after_upload is defined
  changed_when: true
```

Puis dans chaque playbook :

```yaml
- name: Run post-upload commands
  ansible.builtin.import_tasks: tasks/run_post_upload_commands.yml
```


### Pour l'instance 4 : Remplacer par des commandes explicites

```yaml
# AVANT — meme commande repetee deux fois via boucle
- name: Resize ceph-osd
  command: "{{ item }}"
  with_items:
    - "/usr/bin/ceph-bluestore-tool bluefs-bdev-expand --path {{ path }}"
    - "/usr/bin/ceph-bluestore-tool bluefs-bdev-expand --path {{ path }}"

# APRES — une seule commande explicite (supprimer le doublon)
- name: Resize ceph-osd
  ansible.builtin.command:
    cmd: >-
      /usr/bin/ceph-bluestore-tool bluefs-bdev-expand
      --path {{ ceph_expansion_lv_ceph_osd_number.files[0].path }}
  register: resize_result
  changed_when: "'already' not in resize_result.stdout"
```

