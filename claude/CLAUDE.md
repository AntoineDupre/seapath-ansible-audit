```

## Architecture

### Key Configuration Files
- `ansible.cfg` — sets `roles_path`, `library`, plugin paths (including ceph-ansible submodule paths), `forks=20`, `any_errors_fatal=True`, skips `package-install` tag by default
- `ansible-requirements.yaml` — external collections: `community.libvirt`, `community.general`, `containers.podman`, `ansible.posix`, `ansible.utils`, `openstack.config_template`
- `vars/` — distribution-specific path variables (`Debian_paths.yml`, `CentOS_paths.yml`, etc.), included dynamically based on detected distro

### Playbook Entry Points
- `playbooks/seapath_setup_main.yaml` — main entry point; detects distro, runs prerequisite and configuration playbooks
- `playbooks/cluster_setup_cephadm.yaml` — Ceph cluster deployment via cephadm
- `playbooks/deploy_vms_cluster.yaml` / `deploy_vms_standalone.yaml` — VM lifecycle management
- `playbooks/replace_machine_*.yaml` — complex node replacement and cluster shrink operations
- `playbooks/seapath_update_*.yaml` — rolling system updates per distro

### Role Organization (55+ roles)
Roles are organized by concern:

- **Distribution setup:** `debian`, `centos`, `oraclelinux`, `yocto` (and `*_hypervisor`, `*_physical_machine` variants); `detect_seapath_distro` auto-detects at runtime
- **Clustering/HA:** `configure_ha` (Pacemaker + Corosync), `cephadm` (Ceph), `ceph_prepare_installation`
- **Networking (11 roles):** `network_basics`, `network_systemdnetworkd`, `network_netplan`, `network_configovs` (Open vSwitch), `network_sriovpool`, `network_clusternetwork`, `network_buildhosts`, `conntrackd`, `iptables`, etc.
- **Virtualization:** `configure_libvirt`, `deploy_vms_cluster`, `deploy_vms_standalone`, `add_libvirtadmin_user`, `vmmgrapi`
- **Real-time/Timing:** `timemaster` (PTP/NTP), `configure_nic_irq_affinity`
- **Security:** `debian_hardening` (ANSSI BP-28 compliance)
- **Testing/CI:** `deploy_cukinia`, `ci_yocto`, `ci_centos`, `ci_restore_snapshot`, etc.

### Inventory Structure
Inventory groups and their meaning:
- `cluster_machines` — 3 nodes forming the HA cluster
- `hypervisors` — VM-capable hosts
- `osds` / `mons` / `clients` — Ceph roles
- `observers` — optional non-computing cluster nodes
- `standalone_machine` — single-node deployments
- `VMs` — guest VMs to deploy

See `inventories/examples/` for canonical examples.

### ceph-ansible Submodule
The `ceph-ansible/` submodule (branch `stable-8.0`) is patched by `prepare.sh` with patches from `src/ceph-ansible-patches/`. Its `library/`, `plugins/actions`, `plugins/callback`, and `plugins/filter` directories are loaded via `ansible.cfg`. There is also a separate `cephadm` role for the modern containerized Ceph deployment path.

### Custom Module
`library/cluster_vm.py` — custom Ansible module for cluster VM operations; loaded automatically via `ansible.cfg`.

## Linting Rules
`ansible-lint.conf` skips: `yaml` (line length), `role-name`, `risky-file-permissions`, `fqcn-builtins`. Warns but does not fail on: `no-handler`, `no-changed-when`, `no-relative-paths`. The `ceph-ansible/` submodule is excluded from linting.

## CI
GitHub Actions workflows in `.github/workflows/` test against all 4 distributions. PRs require passing `ansible-lint` and a system test report. The CI repository is https://github.com/seapath/ci.
