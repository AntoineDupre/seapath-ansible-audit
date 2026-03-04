Inventaire complet de toutes les instances

### Roles (73 instances)

| Fichier | Ligne | Tache | Categorie |
|---------|-------|-------|-----------|
| `roles/add_libvirtadmin_user/tasks/main.yml` | 39 | Get root user's home directory | Lecture seule, devrait etre `false` |
| `roles/backup_restore/tasks/main.yml` | 29 | Check configuration of backup-restore.sh | Lecture seule, devrait etre `false` |
| `roles/centos/handlers/main.yml` | 20 | Update Grub | Handler, acceptable |
| `roles/centos_hypervisor/tasks/main.yml` | 184 | Load seapath-rt-host tuned profile | Necessite un conditionnel |
| `roles/centos_hypervisor/tasks/main.yml` | 189 | Unload tuned profile | Necessite un conditionnel |
| `roles/centos_physical_machine/handlers/main.yml` | 16 | Rebuild initramfs if necessary | Handler, acceptable |
| `roles/ceph_expansion_lv/tasks/main.yml` | 74 | Resize Ceph OSD with bluestore tool | Necessite un conditionnel |
| `roles/ceph_expansion_lv/tasks/main.yml` | 116 | Resize ceph-osd | Necessite un conditionnel |
| `roles/ceph_expansion_vg/tasks/main.yml` | 49 | Extend partitions | Necessite un conditionnel |
| `roles/ceph_expansion_vg/tasks/main.yml` | 53 | Resize PVs | Necessite un conditionnel |
| `roles/ceph_prepare_installation/tasks/main.yml` | 31 | Refresh LVM VG | Necessite un conditionnel |
| `roles/ceph_prepare_installation/tasks/main.yml` | 40 | Cleanup Ceph LV with ceph-volume | Ponctuel, acceptable |
| `roles/ceph_prepare_installation/tasks/main.yml` | 54 | Cleanup Ceph OSD disks | Ponctuel, acceptable |
| `roles/ceph_prepare_installation/tasks/main.yml` | 60 | Cleanup Ceph lvm's partition | Ponctuel, acceptable |
| `roles/cephadm/tasks/main.yml` | 62 | Install cephadm repository | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 67 | Install cephadm | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 72 | Install ceph-common package | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 165 | Tag ceph image for local registry | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 170 | Push ceph image to local registry | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 217 | Bootstrap Ceph cluster | Ponctuel, garde par `when:` |
| `roles/cephadm/tasks/main.yml` | 283 | Add hosts to Ceph orchestrator | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 331 | Zap the volume on nodes that need OSDs | Ponctuel, acceptable |
| `roles/cephadm/tasks/main.yml` | 345 | Add OSD daemon on nodes | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 369 | Create RBD pool | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 376 | Enable RBD application on the pool | Necessite un conditionnel |
| `roles/cephadm/tasks/main.yml` | 392 | Create CephX user client.libvirt | Garde par `when:` |
| `roles/cephadm/tasks/main.yml` | 400 | Update CephX user client.libvirt | Garde par `when:` |
| `roles/ci_centos/tasks/main.yaml` | 49 | Launch cukinia tests | Tache CI, priorite basse |
| `roles/ci_cleanup_varlog/tasks/main.yml` | 7 | Wipe the /var/log/ directory content | Tache CI, priorite basse |
| `roles/ci_reinstalliso/tasks/main.yml` | 29 | Set next boot entry to DVD-ROM | Ponctuel, acceptable |
| `roles/ci_restore_snapshot/tasks/main.yml` | 21 | Update grub | Type handler, acceptable |
| `roles/ci_restore_snapshot/tasks/main.yml` | 25 | Merge lvm snapshot | Ponctuel, acceptable |
| `roles/ci_restore_snapshot/tasks/main.yml` | 31 | Refresh LVM | Necessite un conditionnel |
| `roles/ci_yocto/clean_arm_machines/tasks/main.yaml` | 21 | Execute Swupdate | Ponctuel, acceptable |
| `roles/ci_yocto/clean_arm_machines/tasks/main.yaml` | 33 | Unmount overlays | Necessite un conditionnel |
| `roles/ci_yocto/run_tests/tasks/main.yaml` | 18 | Launch cukinia tests | Tache CI, priorite basse |
| `roles/configure_ha/tasks/main.yml` | 46 | Generating corosync authkey | Necessite un conditionnel (verifier si le fichier existe) |
| `roles/configure_ha/tasks/main.yml` | 133 | Disable stonith | Necessite un conditionnel |
| `roles/configure_libvirt_rdb_secret/tasks/main.yml` | 59 | Create libvirt rbd secret | Garde par `when:` |
| `roles/configure_libvirt_rdb_secret/tasks/main.yml` | 66 | Set key in libvirt secret | Garde par `when:` |
| `roles/debian/handlers/main.yml` | 24 | Update-grub | Handler, acceptable |
| `roles/debian_grub_bootcount/handlers/main.yml` | 9 | Rebuild initramfs if necessary | Handler, acceptable |
| `roles/debian_grub_bootcount/handlers/main.yml` | 13 | Update-grub | Handler, acceptable |
| `roles/debian_grub_bootcount/tasks/main.yml` | 174 | Create /boot/efi/bootcountenv | Necessite un conditionnel |
| `roles/debian_hardening/handlers/main.yml` | 7 | Update sysfs values using sysctl | Handler, acceptable |
| `roles/debian_hardening/tasks/main.yml` | 383 | Update grub | Necessite un conditionnel |
| `roles/debian_hypervisor/tasks/main.yml` | 171 | Load seapath-rt-host tuned profile | Necessite un conditionnel |
| `roles/debian_hypervisor/tasks/main.yml` | 176 | Unload tuned profile | Necessite un conditionnel |
| `roles/debian_physical_machine/handlers/main.yml` | 16 | Rebuild initramfs if necessary | Handler, acceptable |
| `roles/deploy_cephfs/tasks/main.yml` | 28 | Create CephFS volume | Necessite un conditionnel |
| `roles/deploy_python3_setup_ovs/tasks/main.yml` | 17 | Install python3-setup-ovs | Necessite un conditionnel |
| `roles/deploy_vm_manager/tasks/main.yml` | 17 | Install vm_manager | Necessite un conditionnel |
| `roles/deploy_vms_standalone/tasks/main.yml` | 27 | Undefine VM | Necessite un conditionnel |
| `roles/iptables/handlers/main.yml` | 9 | Reload iptables rules | Handler, acceptable |
| `roles/network_clusternetwork/tasks/main.yml` | 20-35 | Configuration bridge OVS (6 taches) | Necessite un conditionnel |
| `roles/network_configovs/tasks/main.yml` | 33 | Run extra ovs-vsctl commands | Necessite un conditionnel |
| `roles/network_netplan/handlers/main.yml` | 3 | Apply Netplan configuration | Handler, acceptable |
| `roles/network_sriovpool/tasks/main.yml` | 23 | Create libvirt SRIOV network pool | Necessite un conditionnel |
| `roles/network_sriovpool/tasks/main.yml` | 26 | AutoStart libvirt SRIOV network pool | Necessite un conditionnel |
| `roles/network_sriovpool/tasks/main.yml` | 29 | Start libvirt SRIOV network pool | Necessite un conditionnel |
| `roles/oraclelinux/handlers/main.yml` | 18 | Update-grub | Handler, acceptable |
| `roles/oraclelinux_physical_machine/handlers/main.yml` | 16 | Rebuild initramfs if necessary | Handler, acceptable |
| `roles/update/tasks/main.yaml` | 27 | Set hypervisor in maintenance mode | Necessite un conditionnel |
| `roles/update/tasks/main.yaml` | 43 | Apply update | Ponctuel, acceptable |
| `roles/update/tasks/main.yaml` | 64 | Unset maintenance mode | Necessite un conditionnel |
| `roles/yocto/kernel_params/tasks/main.yaml` | 27 | Mount the boot partition | Necessite un conditionnel |
| `roles/yocto/kernel_params/tasks/main.yaml` | 36 | Umount the boot partition | Necessite un conditionnel |

### Playbooks (31 instances)

| Fichier | Ligne | Tache | Categorie |
|---------|-------|-------|-----------|
| `playbooks/ci_prepare_VMs.yaml` | 32 | Load tuned profile | Necessite un conditionnel |
| `playbooks/cluster_setup_ceph.yaml` | 59 | Disable Ceph mgr restful module | Necessite un conditionnel |
| `playbooks/cluster_setup_ceph.yaml` | 68 | Disable rbd lock | Necessite un conditionnel |
| `playbooks/replace_machine_remove_machine_cephadm.yaml` | 27 | Remove machine from pacemaker | Ponctuel, acceptable |
| `playbooks/replace_machine_remove_machine_cephadm.yaml` | 33 | Remove machine from ceph cluster | Ponctuel, acceptable |
| `playbooks/replace_machine_shrink_mgr.yaml` | 62 | Save mgr dump output | Lecture seule, devrait etre `false` |
| `playbooks/replace_machine_shrink_mgr.yaml` | 112 | Ensure mgr is stopped | Lecture seule, devrait etre `false` |
| `playbooks/replace_machine_shrink_osd.yaml` | 199 | Close dmcrypt close on devices | Necessite un conditionnel |
| `playbooks/replace_machine_shrink_osd.yaml` | 223 | Use ceph-volume lvm zap | Ponctuel, acceptable |
| `playbooks/replace_machine_shrink_pacemaker.yaml` | 16 | Remove pacemaker node | Ponctuel, acceptable |
| `playbooks/seapath_update_debian.yaml` | 26 | Disable password protection | Necessite un conditionnel |
| `playbooks/seapath_update_debian.yaml` | 31 | Enable Grub Boot Count | Necessite un conditionnel |
| `playbooks/seapath_update_debian.yaml` | 53 | Remove snapshot if success | Ponctuel, acceptable |
| `playbooks/seapath_update_debian.yaml` | 63 | Enable grub password protection | Necessite un conditionnel |
| `playbooks/seapath_update_yocto_cluster.yaml` | 14 | Set hypervisor in maintenance mode | Necessite un conditionnel |
| `playbooks/seapath_update_yocto_cluster.yaml` | 30 | Apply update | Ponctuel, acceptable |
| `playbooks/seapath_update_yocto_cluster.yaml` | 49 | Unset maintenance mode | Necessite un conditionnel |
| `playbooks/seapath_update_yocto_standalone.yaml` | 19 | Apply update | Ponctuel, acceptable |
| `playbooks/test_run_cukinia.yaml` | 42 | Run tests | Tache CI, priorite basse |
| `playbooks/test_run_cukinia.yaml` | 65 | Run tests cluster test | Tache CI, priorite basse |
