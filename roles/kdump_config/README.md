# Role: `kdump_config`

Installs, configures, and enables the Linux kdump crash dump service with support for local, NFS, SSH, and raw device dump targets, plus kernel panic trigger tuning.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` (for `sysctl` module)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `kdump_config_dump_target` | `"Local"` | Dump target type: `Local`, `NFS`, `SSH`, or `Raw` |
| `kdump_config_local_path` | `"/var/crash"` | Filesystem path for local crash dumps |
| `kdump_config_nfs_server` | `"nfs.example.com:/remote/crash/path"` | NFS server and export path (used when target is `NFS`) |
| `kdump_config_ssh_server` | `"ssh.example.com"` | SSH server hostname (used when target is `SSH`) |
| `kdump_config_ssh_user` | `"root"` | SSH user for remote crash dump (used when target is `SSH`) |
| `kdump_config_raw_device` | `"/dev/sdb1"` | Block device for raw dump (used when target is `Raw`) |
| `kdump_config_dump_level` | `31` | makedumpfile `-d` level (0-31); 31 excludes all unnecessary pages |
| `kdump_config_failure_action` | `"reboot"` | Action on dump failure: `reboot`, `halt`, `poweroff`, `shell`, or `dump_to_rootfs` |
| `kdump_config_panic_triggers.system_hang` | `true` | Enable kernel panic on system hang (SysRq, NMI) |
| `kdump_config_panic_triggers.oom_killer` | `true` | Enable kernel panic on OOM killer invocation |
| `kdump_config_panic_triggers.soft_lockup` | `true` | Enable kernel panic on soft lockup detection |
| `kdump_config_panic_triggers.process_d_state` | `true` | Enable kernel panic on hung task (D-state process) |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Configure kdump kernel crash dump service
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: kdump_config
      vars:
        kdump_config_dump_target: "Local"
        kdump_config_local_path: "/var/crash"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
