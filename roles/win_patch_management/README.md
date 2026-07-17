# Role: `win_patch_management`

Manages Windows Update patching with support for installing all updates, targeting specific KBs, or generating an HTML report of available patches.

## Requirements

- ansible-core 2.15+
- Collection `ansible.windows`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `win_patch_management_mode` | `install_all` | Operation mode: `install_all`, `install_kb`, or `report` |
| `win_patch_management_kb_list` | `[]` | List of specific KB article IDs to install (used with `install_kb` mode) |
| `win_patch_management_category` | `*` | Windows Update category filter (e.g. `SecurityUpdates`, `*` for all) |
| `win_patch_management_reboot` | `true` | Whether to automatically reboot after installing updates |
| `win_patch_management_report_path` | `C:\ansible_reports` | Directory to store patch report files |
| `win_patch_management_report_dest` | `C:\inetpub\wwwroot\win_patch_report.html` | Full path for the generated HTML report |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Manage Windows patches and updates
  hosts: all_windows
  gather_facts: true
  roles:
    - role: win_patch_management
      vars:
        win_patch_management_mode: report
        win_patch_management_reboot: false
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
