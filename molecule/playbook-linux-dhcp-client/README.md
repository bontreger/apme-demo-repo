# Molecule: `playbook-linux-dhcp-client`

Tests DHCP client configuration aligned with `playbooks/linux-dhcp-client.yml`.

## Controller setup (Amazon Linux / non-RHEL)

The playbook uses `redhat.rhel_system_roles.network`. On a non-RHEL Molecule
controller, install the upstream collection:

```bash
ansible-galaxy collection install fedora.linux_system_roles --no-deps
```

`converge.yml` mirrors the playbook logic but calls `fedora.linux_system_roles.network`.

## Run

```bash
export AWS_REGION=us-east-2
molecule test -s playbook-linux-dhcp-client
```

`converge.yml` always builds `network_connections` from the primary IPv4 NIC
(same as the playbook's auto-detect path). Only `rhel9` is provisioned — the
playbook rejects RHEL 10 at its OS assert.

## AAP

Build the custom EE from `execution_environments/` (includes
`redhat.rhel_system_roles`) and assign it to `ACL_linux_dhcp_client`. The
platform Default EE does not include this collection.
