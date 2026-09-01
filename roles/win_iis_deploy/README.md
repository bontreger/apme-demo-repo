# Role: `win_iis_deploy`

**Tags:** pillar:os,lifecycle:day1,maturity:l2-process-oriented,platform:windows,tech:iis

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
| `win_iis_deploy_reboot` | `false` | Reboot when installing the Web-Server feature requires it |

## Behaviour

- **The role deploys to the Default Web Site.** It installs the feature, starts
  W3SVC and writes an index page into the web root; it does not create sites or
  bindings.
- **The health check runs on the guest** against `http://localhost/`, so it
  reports on IIS rather than on the network path between the controller and the
  host. It is skipped in check mode, where IIS may not be installed yet.
- **Opening HTTPS does not configure HTTPS.** With
  `win_iis_deploy_firewall_https` enabled the role opens port 443, but it
  creates no HTTPS binding and installs no certificate. Set it to `false`
  unless something else configures the binding.
- **A custom `win_iis_deploy_webroot` must already exist.** The default works
  because IIS creates it.
- **Reboots are opt-in.** Installing the feature can require a reboot; with
  `win_iis_deploy_reboot` left `false` the role continues without rebooting,
  and the tasks after it may fail until the host is restarted.

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
