# Role: `dns_sync`

**Tags:** pillar:network,lifecycle:day2,maturity:l2-process-oriented,platform:linux,tech:networkmanager  
**Supported OS:** RHEL 7/8/9/10

Configures DNS resolver settings on target hosts via NetworkManager. Nameservers
and search domains are provided as variables, ensuring the role works correctly
when executed from AAP (where the Execution Environment's own `/etc/resolv.conf`
is not a viable source).

Changes are applied through `nmcli` so they persist across reboots and
NetworkManager restarts.

## Requirements

- `community.general` collection (for the `nmcli` module).
- NetworkManager must be active on the target hosts (standard on RHEL 7+).

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `dns_sync_nameservers` | `[]` | **Required.** List of DNS nameserver IPs (e.g. `["10.0.0.1", "10.0.0.2"]`) |
| `dns_sync_search_domains` | `[]` | List of DNS search domains (e.g. `["example.com"]`) |
| `dns_sync_connection` | `""` | NetworkManager connection name. Auto-detected from the first active connection when empty |
| `dns_sync_restart_connection` | `true` | Restart the NM connection after changes so `/etc/resolv.conf` is updated immediately |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Configure DNS resolvers across fleet
  hosts: all_rhel
  become: true
  roles:
    - role: dns_sync
      vars:
        dns_sync_nameservers:
          - "10.0.0.1"
          - "10.0.0.2"
        dns_sync_search_domains:
          - "example.com"
          - "corp.local"
```

### Override the NetworkManager connection name

```yaml
- role: dns_sync
  vars:
    dns_sync_nameservers: ["10.0.0.1"]
    dns_sync_connection: "ens192"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
