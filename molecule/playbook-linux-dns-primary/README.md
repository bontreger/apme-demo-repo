# Molecule: `playbook-linux-dns-primary`

Tests `playbooks/linux-dns-primary.yml` and the in-repo `bind_primary_dns` role.

## Controller setup

No extra Galaxy collections beyond the standard molecule stack
(`molecule/requirements.yml`). The role uses `ansible.posix` (firewalld), which
is in `collections/requirements.yml`.

## Run

```bash
export AWS_REGION=us-east-2
molecule test -s playbook-linux-dns-primary
```

Only `rhel9` is provisioned — the playbook OS assert allows 7/8/9 only (RHEL 10
is tracked in #60).

## Molecule-specific overrides

| Override | Reason |
|----------|--------|
| `files/named-primary.conf` / `files/zones/` | Stock role samples use `192.168.62.0/24`; molecule VPC is `10.77.0.0/24` (paths via `MOLECULE_SCENARIO_DIRECTORY` in inventory) |
| `bind_primary_dns_bind_manage_firewall: false` | Cloud AMIs often lack `python3-firewall` until explicitly installed |

## Verify scope

Verify asserts package install, deployed `named.conf` and zone files, and that
`named` is **enabled**. Service **active** state is reported for peer review but
not required to pass.

## Peer-review notes

- Production sample configs in `roles/bind_primary_dns/files/` still target
  `192.168.62.0/24` — document for lab/AAP users. Molecule supplies VPC-aligned
  configs via `MOLECULE_SCENARIO_DIRECTORY`.
- Role variable `bind_primary_dns_bind_config_src` defaults to `named-primary.conf`
  for backward compatibility.

## AAP

Template: `ACL_linux_dns_bind_primary`. Inventory group: `primary_dns`. Default
EE. Consider `bind_primary_dns_bind_manage_firewall: false` on lab hosts.
