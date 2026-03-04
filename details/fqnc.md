
## Probleme

Le depot contient des noms de modules non pleinement qualifies. Le fichier `ansible-lint.conf` desactive actuellement la regle `fqcn-builtins`, ce qui signifie que des noms de modules nus comme `command:`, `copy:`, `file:` sont utilises au lieu de `ansible.builtin.command:`, `ansible.builtin.copy:`, `ansible.builtin.file:`.

Cela a de l'importance car :

- **Avertissements de deprecation Ansible 2.17+** — les noms nus genereront des avertissements et pourront casser dans les futures versions
- **Collisions d'espaces de noms** — un `user:` nu pourrait resoudre vers un module communautaire au lieu de `ansible.builtin.user` si une collection en fournit un avec le meme nom
- **Lisibilite** — le FQCN rend immediatement clair si un module est builtin, de `community.general`, `ansible.posix`, etc.
- **Bonne pratique** — la communaute Ansible recommande le FQCN pour tout code de production

## Etat actuel

Ligne de `ansible-lint.conf` qui desactive cette regle :

```yaml
skip_list:
  - fqcn-builtins
```

### Comptage d'utilisation des modules


- `command`: `ansible.builtin.command` 
- `group`: `ansible.builtin.group` 
- `file`: `ansible.builtin.file` |
- `lineinfile`: `ansible.builtin.lineinfile` 
- `set_fact`: `ansible.builtin.set_fact` 
- `template`: `ansible.builtin.template` 
- `copy`: `ansible.builtin.copy` 
- `shell`: `ansible.builtin.shell` 
- `user`: `ansible.builtin.user` 
- `service`: `ansible.builtin.service` 
- `debug`: `ansible.builtin.debug` 
- `include_vars`: `ansible.builtin.include_vars` 
- `replace`: `ansible.builtin.replace` 
- `systemd`: `ansible.builtin.systemd_service` 
- `fetch`:`ansible.builtin.fetch` 
- `stat`: `ansible.builtin.stat` 
- `apt`: `ansible.builtin.apt` 
- `getent`: `ansible.builtin.getent` 
- `get_url`: `ansible.builtin.get_url` 

Certaines taches melangent des noms nus et FQCN au sein du meme fichier est deroutant.

## Comment corriger

### Etape 1 : Activer la regle de lint

Dans `ansible-lint.conf`, supprimer `fqcn-builtins` de la liste d'exclusion :

```yaml
skip_list:
  # Supprimer : - fqcn-builtins
  - yaml
  - role-name
  - risky-file-permissions
```


### Etape 2 : Migration automatisee

La plupart des remplacements sont mecaniques et peuvent etre corrigés via:
```bash
ansible-lint -c anslible-lint.conf --fix
```

**Mises en garde importantes :**
- Exclure le sous-module `ceph-ansible/` (deja exclu du linting)
- Verifier les remplacements de `systemd:`  `ansible.builtin.systemd_service` est le nom moderne mais `ansible.builtin.systemd` fonctionne aussi
- Le remplacement de `group:` doit faire attention a ne pas correspondre aux cles YAML comme `group:` a l'interieur des parametres des modules `file:` ou `copy:`

### Etape 3 : Revue manuelle

Apres le remplacement automatise, verifier manuellement :
- Pas de faux positifs (ex. `group:` en tant que parametre de `copy:` ou `file:`)
- Les fichiers de handlers sont egalement mis a jour
- Les fichiers de templates et de variables ne sont pas affectes

### Etape 4 : Executer ansible-lint pour detecter les oublis

```bash
ansible-lint -c ansible-lint.conf
```

Corriger tout nom de module nu restant.

