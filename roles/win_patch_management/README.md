# Role: `win_patch_management`

**Tags:** pillar:os,lifecycle:day2,maturity:l2,platform:windows,tech:patching

Manages Windows Update patching with support for installing all updates, targeting specific KBs, or generating an HTML report of available patches.

## Requirements

- ansible-core 2.15+
- Collection `ansible.windows`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `win_patch_management_mode` | `report` | Operation mode: `report`, `install_all`, or `install_kb` |
| `win_patch_management_kb_list` | `[]` | Update titles or KB numbers, matched as regular expressions. Required by `install_kb` |
| `win_patch_management_category` | `*` | Windows Update category filter (e.g. `SecurityUpdates`, `*` for all) |
| `win_patch_management_reboot` | `true` | Reboot automatically when updates require it. Applies to both installing modes |
| `win_patch_management_report_path` | `C:\ansible_reports` | Base directory for reports; the default `report_dest` is placed here |
| `win_patch_management_report_dest` | `{{ win_patch_management_report_path }}\win_patch_report.html` | Full path for the generated HTML report. Its parent directory is created |

## Behaviour

- **`report` is the default mode** because it is the only read-only one.
  `install_all` and `install_kb` patch the host and may reboot it, so they have
  to be asked for explicitly.
- **`install_kb` does nothing when `win_patch_management_kb_list` is empty.**
  An empty accept list matches *every* update in `win_updates`, so an empty
  list would install everything rather than the specific KBs the mode implies.
- **Reports are not idempotent.** The HTML is rewritten on every run, so report
  mode always reports `changed`.
- **An unrecognised mode does nothing** and the run reports success.
- **Patching can take hours.** `win_updates` runs the Windows Update API on the
  host; set job timeouts accordingly.

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
