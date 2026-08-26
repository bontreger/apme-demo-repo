# Role: `nfs_server`

**Tags:** pillar:os,lifecycle:day1,maturity:l2-process-oriented,platform:linux,tech:nfs  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Installs NFS utilities, creates a shared directory, configures exports, manages the nfs-server service, and opens the firewall for NFS access.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` (for `firewalld` module)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `nfs_server_shared_directory` | `"/srv/myshare"` | Absolute path of the directory to share via NFS |
| `nfs_server_export_entries` | `["client1.example.com(rw,no_root_squash)"]` | List of client access entries appended to the export line |
| `nfs_server_exports_filename` | `"ansible-share.exports"` | Filename placed in `/etc/exports.d/` for the export definition |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Deploy and configure NFS server
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: nfs_server
      vars:
        nfs_server_shared_directory: "/srv/myshare"
        nfs_server_export_entries:
          - "client1.example.com(rw,no_root_squash)"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
