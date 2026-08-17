# Molecule: `playbook-linux-dns-secondary`

Tests `playbooks/linux-dns-secondary.yml` and the in-repo `bind_secondary_dns` role.

## Controller setup

No extra Galaxy collections beyond the standard molecule stack
(`molecule/requirements.yml`). The role uses `ansible.posix` (firewalld), which
is in `collections/requirements.yml`.

## Run

```bash
export AWS_REGION=us-east-2
molecule test -s playbook-linux-dns-secondary
```

Only `rhel9` is provisioned — the playbook OS assert allows 7/8/9 only (RHEL 10
is tracked in #60).

## Molecule-specific overrides

| Override | Reason |
|----------|--------|
| `files/named-secondary.conf` | Stock sample points masters at `192.168.62.10`; molecule uses `10.77.0.10` |
| `bind_secondary_dns_bind_manage_firewall: false` | Cloud AMIs often lack `python3-firewall` until explicitly installed |

## Verify scope

Verify asserts package install, deployed `named.conf`, and that `named` is
**enabled**. Service **active** state is reported for peer review but not
required to pass (slave zone transfer needs a primary; not provisioned in this
scenario).

## Peer-review notes

- Production sample config in `roles/bind_secondary_dns/files/` still targets
  primary `192.168.62.10` — document for lab/AAP users.
- Role variable `bind_secondary_dns_bind_config_src` defaults to
  `named-secondary.conf` for backward compatibility.

## AAP

Template: `ACL_linux_dns_bind_secondary`. Inventory group: `secondary_dns`.
Default EE. Consider `bind_secondary_dns_bind_manage_firewall: false` on lab
hosts.
