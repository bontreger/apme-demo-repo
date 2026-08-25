# Role: `selinux_config`

**Tags:** pillar:security,lifecycle:day1,maturity:l2,platform:linux,tech:selinux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Configures SELinux runtime mode and persistent state on RHEL systems, with input validation, automatic rollback on failure, and a post-change reboot notification.

## Requirements

- ansible-core 2.15+
- `ansible.posix` collection

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `selinux_config_permanent_state` | `disabled` | Desired SELinux state written to the config file (`enforcing`, `permissive`, or `disabled`) |
| `selinux_config_runtime_state` | `permissive` | Desired SELinux runtime mode applied immediately |
| `selinux_config_path` | `/etc/selinux/config` | Path to the SELinux configuration file |
| `selinux_config_valid_states` | `["enforcing", "permissive", "disabled"]` | Allowed values for state validation |
| `selinux_config_reboot_required_msg` | *(multi-line reboot notice)* | Message displayed when a reboot is required to apply changes |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Configure SELinux state across network
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: selinux_config
      vars:
        selinux_config_permanent_state: enforcing
        selinux_config_runtime_state: enforcing
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
