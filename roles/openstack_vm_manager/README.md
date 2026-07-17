# Role: `openstack_vm_manager`

Manages OpenStack virtual machine lifecycle operations including creation, deletion, power management, resizing, snapshots, volume and floating IP management, network interfaces, and security groups.

## Requirements

- ansible-core 2.15+
- `openstack.cloud` collection
- OpenStack authentication credentials provided via `vault_os_auth`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `openstack_vm_manager_vm_list` | `[]` | List of VM definitions to manage (structure depends on the operation) |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Run OpenStack vm lifecycle automation tool kit
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: openstack_vm_manager
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
