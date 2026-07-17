# Role: `hosts_config`

Standardizes /etc/hosts across target hosts by fetching a template from a designated template server, merging it, and ensuring the local host entry is present.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- A `template_servers` inventory group or `template_server` variable defining the source host

## Role Variables

This role does not define variables in `defaults/main.yml`. It relies on the following inventory/group variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `template_server.hosts_file` | *(required)* | Path to the hosts template file on the template server |
| `template_server.host` | *(required)* | Hostname of the template server (fallback if group is empty) |
| `backup.enabled` | `true` | Whether to create backups before modifying files |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL9 System Standardization Configuration
  hosts: rhel9
  become: true
  roles:
    - role: hosts_config
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
