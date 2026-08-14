# Molecule: `playbook-linux-dns-sync`

Tests `playbooks/linux-dns-sync.yml` and the in-repo `dns_sync` role.

## Controller setup

No extra Galaxy collections beyond the standard molecule stack. The role copies
a resolver file from the **molecule controller** (`delegate_to: localhost`) to
each target host.

## Run

```bash
export AWS_REGION=us-east-2
molecule test -s playbook-linux-dns-sync
```

Only `rhel9` is provisioned — playbook OS assert allows 7/8/9 only (RHEL 10 in
#60).

## Molecule-specific overrides

| Variable | Value | Reason |
|----------|-------|--------|
| `dns_sync_source_file_path` | `MOLECULE_SCENARIO_DIRECTORY/files/resolv.conf` | Known-good source on controller |
| `dns_sync_dest_file_path` | `/tmp/molecule-dns-sync-resolv.conf` | Avoid overwriting live `/etc/resolv.conf` on EC2 |

## AAP

Template: `ACL_linux_dns_sync`. Inventory: `all_rhel`. Default EE. Source
defaults to controller `/etc/resolv.conf` unless overridden at launch.
