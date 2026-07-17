# Role: `yum_repo_config`

Configures a local YUM/DNF repository server from an ISO image and registers client hosts to consume packages from it over HTTP or HTTPS.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- Supported only on RHEL 8 and RHEL 9.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `yum_repo_config_protocol` | `http` | Protocol for repo access (`http` or `https`) |
| `yum_repo_config_source` | `iso` | Repository source type |
| `yum_repo_config_document_root` | `/var/www/html` | Web server document root for hosting the repository |
| `yum_repo_config_iso_path` | `/data/rhel-8.10-x86_64-dvd.iso` | Path to the RHEL ISO file on the server |
| `yum_repo_config_release` | `8.10` | RHEL release version string |
| `yum_repo_config_relative_path` | `RHEL-8.10/RedHatEnterpriseLinux/x86_64` | Relative path within the document root for repository content |
| `yum_repo_config_mount_path` | `/mnt/RHEL-8.10/RedHatEnterpriseLinux/x86_64` | Mount point for the ISO filesystem |
| `yum_repo_config_repo_name` | `remote_repo_{{ yum_repo_config_release }}-{{ yum_repo_config_protocol }}` | Repository name used in yum repo configuration files |
| `yum_repo_config_server_group` | `{{ repo_server_group \| default('server') }}` | Inventory group name identifying repo server hosts |
| `yum_repo_config_client_group` | `{{ repo_client_group \| default('client') }}` | Inventory group name identifying repo client hosts |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Setup YUM repository server
  hosts: server
  become: true
  roles:
    - role: yum_repo_config
      vars:
        yum_repo_config_mode: server
        yum_repo_config_iso_path: /data/rhel-9.4-x86_64-dvd.iso
        yum_repo_config_release: "9.4"

- name: Setup YUM repository clients
  hosts: client
  become: true
  roles:
    - role: yum_repo_config
      vars:
        yum_repo_config_mode: client
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
