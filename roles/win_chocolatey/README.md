# Role: `win_chocolatey`

**Tags:** pillar:os,lifecycle:day1,maturity:l2-process-oriented,platform:windows,tech:chocolatey

Installs the Chocolatey package manager on Windows hosts and manages packages through it.

## Requirements

- ansible-core 2.15+
- Collection `chocolatey.chocolatey`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `win_chocolatey_state` | `present` | Desired state of managed packages (`present`, `absent`, `latest`) |
| `win_chocolatey_packages` | `[]` | List of Chocolatey package names to manage |
| `win_chocolatey_version` | `1.4.0` | Version of Chocolatey itself to install |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Manage packages via Chocolatey
  hosts: all_windows
  gather_facts: false
  roles:
    - role: win_chocolatey
      vars:
        win_chocolatey_packages:
          - git
          - vscode
          - 7zip
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
