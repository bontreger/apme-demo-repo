# Role: `networkmanager_cli`

**Tags:** pillar:network,lifecycle:day2,maturity:l2,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Manages NetworkManager connections and devices via `nmcli`, supporting view, add, modify, activate/deactivate, and delete operations.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `networkmanager_cli_nm_action_view` | `false` | Display current NetworkManager connection and device information |
| `networkmanager_cli_nm_connections_to_add` | `[]` | List of connection definitions to create (each with `name`, `ifname`, `type`, `ip4`, etc.) |
| `networkmanager_cli_nm_connections_to_modify` | `[]` | List of connection modification definitions to apply |
| `networkmanager_cli_nm_connections_to_activate` | `[]` | List of connection names to activate |
| `networkmanager_cli_nm_devices_to_deactivate` | `[]` | List of device names to deactivate |
| `networkmanager_cli_nm_connections_to_delete` | `[]` | List of connection names to delete |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Manage NetworkManager connections and devices
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: networkmanager_cli
      vars:
        networkmanager_cli_nm_connections_to_add:
          - name: "static-eth0"
            ifname: "eth0"
            type: "ethernet"
            ip4: "192.168.1.100/24"
            gw4: "192.168.1.1"
            method: "manual"
        networkmanager_cli_nm_connections_to_activate:
          - "static-eth0"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
