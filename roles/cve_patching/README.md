# Role: `cve_patching`

**Tags:** pillar:os,lifecycle:day2,maturity:l2,platform:linux,tech:patching  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Queries and optionally applies security patches for a specific CVE ID on RHEL 7/8/9 systems, with reboot requirement detection.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- RHEL subscription with access to security errata repositories

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `cve_patching_cve_id` | `""` | CVE ID to query and patch (e.g. CVE-2023-42753) |
| `cve_patching_apply_patch` | `"yes"` | Whether to apply patches: "yes" or "no" |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Install security patches based on CVE ID
  hosts: all_rhel
  become: true
  roles:
    - role: cve_patching
      vars:
        cve_patching_cve_id: "CVE-2023-42753"
        cve_patching_apply_patch: "yes"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
