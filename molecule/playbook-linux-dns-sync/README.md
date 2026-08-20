# Molecule: `playbook-linux-dns-sync`

Tests `playbooks/linux-dns-sync.yml` and the in-repo `dns_sync` role.

## What it tests

The role configures DNS nameservers and search domains on the target host via
NetworkManager (`nmcli`). The molecule inventory supplies `dns_sync_nameservers`
and `dns_sync_search_domains` as variables. Verify checks that `/etc/resolv.conf`
contains the expected entries after the connection is restarted.

## Run

```bash
export AWS_REGION=us-east-2
molecule test -s playbook-linux-dns-sync
```

## Molecule-specific overrides

| Variable | Value | Reason |
|----------|-------|--------|
| `dns_sync_nameservers` | `["10.77.0.2"]` | Known nameserver for verification |
| `dns_sync_search_domains` | `["molecule.test"]` | Known search domain for verification |

## AAP

Template: `ACL_linux_dns_sync`. Inventory: `all_rhel`. Default EE. Pass
`dns_sync_nameservers` as extra vars at launch.
