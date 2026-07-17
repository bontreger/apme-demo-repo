# Role: `win_software_management`

Installs or removes Windows software packages (MSI/EXE) using `win_package` with support for custom arguments and product IDs.

## Requirements

- ansible-core 2.15+
- Collection `ansible.windows`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `win_software_management_state` | `present` | Desired state of packages (`present` or `absent`) |
| `win_software_management_packages` | `[]` | List of package dictionaries with `name`, `path`, `product_id`, and optional `arguments` |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Manage Windows software packages
  hosts: all_windows
  gather_facts: false
  roles:
    - role: win_software_management
      vars:
        win_software_management_packages:
          - name: 7-Zip
            path: https://www.7-zip.org/a/7z2301-x64.msi
            product_id: "{23170F69-40C1-2702-2301-000001000000}"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
