# Role: `vnc_config`

**Tags:** pillar:os,lifecycle:day1,maturity:l2,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Installs and configures TigerVNC server with user mapping, desktop session selection, password setup, and firewall rules on RHEL systems.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- Collection `ansible.posix` (for `firewalld` module).

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `vnc_config_user` | `admin01` | System user to configure VNC access for |
| `vnc_config_geometry` | `1280x1024` | VNC display resolution (WIDTHxHEIGHT) |
| `vnc_config_display_number` | `2` | VNC display number (1-99); TCP port = 5900 + this value |
| `vnc_config_desktop_session` | `gnome` | Desktop session type to launch (e.g. gnome, xfce) |
| `vnc_config_password` | `""` | VNC connection password (minimum 6 characters; vault-encrypt in production) |
| `vnc_config_packages` | `["tigervnc-server", "@workstation"]` | Packages to install for VNC and desktop environment |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Configure TigerVNC server
  hosts: all_rhel
  become: true
  roles:
    - role: vnc_config
      vars:
        vnc_config_user: admin01
        vnc_config_password: "{{ vault_vnc_password }}"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
