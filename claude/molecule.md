# Add Molecule Tests for a Role

Generate Molecule test scenarios for the role specified by the user: $ARGUMENTS

## Instructions

1. **Read the role's tasks** (`roles/<role_name>/tasks/main.yml` and any included files) and handlers (`roles/<role_name>/handlers/main.yml`) to understand what the role does.

2. **Identify distinct code paths** (e.g. conditionals with `when`, mutually exclusive variables). Each distinct code path should become its own Molecule scenario. The first/main scenario must be named `default`.

3. **Follow the project's established Molecule pattern** from `roles/network_basics/molecule/`. Every scenario needs these 4 files:

### `molecule.yml`
```yaml
---
dependency:
  name: galaxy

driver:
  name: docker

platforms:
  - name: instance
    image: geerlingguy/docker-debian12-ansible:latest
    privileged: true

provisioner:
  name: ansible
  config_options:
    defaults:
      roles_path: "${MOLECULE_PROJECT_DIRECTORY}/.."
  playbooks:
    prepare: prepare.yml
    converge: converge.yml
    verify: verify.yml

verifier:
  name: ansible
```

### `prepare.yml`
- Use **two plays** when the role uses `delegate_to: localhost` or `copy src=` / `template src=`:
  - First play targeting `hosts: localhost` to create source files on the controller.
  - Second play targeting `hosts: all` to prepare the container (install packages, create directories).
- **Install any system packages** that the role or its handlers depend on (e.g. `iptables`, `systemd`, etc.) using `ansible.builtin.apt` with `update_cache: true`.
- **Create required directories** that the role expects to exist.
- When creating Jinja2 template files via `ansible.builtin.copy content=`, escape any `{{ variable }}` placeholders as `{{ '{{ variable }}' }}` so Ansible writes the literal Jinja2 syntax instead of trying to evaluate it.

### `converge.yml`
```yaml
---
- name: Converge
  hosts: all
  gather_facts: false
  roles:
    - role: <role_name>
  vars:
    <variables needed for this scenario>
```

### `verify.yml`
- Use `ansible.builtin.stat` to check file existence, mode, and ownership.
- Use `ansible.builtin.slurp` + `b64decode` filter to read and assert on file content.
- Use `ansible.builtin.assert` with clear `fail_msg` for every assertion.
- For template scenarios, assert that rendered values appear AND that raw Jinja2 placeholders do NOT appear.

## Checklist

- [ ] Read the role's tasks and handlers completely
- [ ] Identify all code paths / scenarios to test
- [ ] Create `roles/<role_name>/molecule/<scenario>/` directories
- [ ] Write all 4 files per scenario
- [ ] Ensure system dependencies are installed in prepare.yml
- [ ] Ensure Jinja2 placeholders in copy content are properly escaped
- [ ] Verify the tests can run with `cd roles/<role_name> && molecule test --all`
