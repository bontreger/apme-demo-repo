# Role: `win_iis_deploy`

Installs IIS Web Server on Windows, configures firewall rules for HTTP/HTTPS, starts the W3SVC service, and deploys a default index page.

## Requirements

- ansible-core 2.15+
- Collection `ansible.windows`
- Collection `community.windows`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `win_iis_deploy_firewall_http` | `true` | Whether to open firewall for HTTP (port 80) |
| `win_iis_deploy_firewall_https` | `true` | Whether to open firewall for HTTPS (port 443) |
| `win_iis_deploy_index_content` | `<h1>IIS is live via Ansible</h1>` | HTML content for the default index page |
| `win_iis_deploy_webroot` | `C:\inetpub\wwwroot` | Path to the IIS web root directory |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Deploy and configure IIS
  hosts: all_windows
  gather_facts: false
  roles:
    - role: win_iis_deploy
      vars:
        win_iis_deploy_index_content: "<h1>Welcome to our site</h1>"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
