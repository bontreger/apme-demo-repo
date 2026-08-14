# Molecule: `playbook-linux-dhcp-server`

Tests `playbooks/linux-dhcp-server.yml` and the in-repo `dhcp_server` role.

## Controller setup

No extra Galaxy collections beyond the standard molecule stack
(`molecule/requirements.yml`). The role uses `ansible.posix` (firewalld), which
is in `collections/requirements.yml`.

## Run

```bash
export AWS_REGION=us-east-2
molecule test -s playbook-linux-dhcp-server
```

Only `rhel9` is provisioned — the playbook OS assert allows 8/9 only (RHEL 10
is tracked in #60).

## Molecule-specific overrides

| Override | Reason |
|----------|--------|
| `files/dhcpd.conf` / `dhcpd6.conf` | Stock role samples use `192.168.0.0/24`; molecule VPC is `10.77.0.0/24` (paths via `MOLECULE_SCENARIO_DIRECTORY` in inventory — `import_playbook` would resolve `playbook_dir` to `playbooks/`) |
| `dhcp_server_dhcp_manage_firewall: false` | Cloud AMIs often lack `python3-firewall` until explicitly installed |

## Verify scope

Verify asserts package install, deployed config files, and that `dhcpd` /
`dhcpd6` are **enabled**. Service **active** state is reported for peer review
but not required to pass.

## Peer-review notes

- Production sample configs in `roles/dhcp_server/files/` still target
  `192.168.0.0/24` — document for lab/AAP users. Molecule supplies VPC-aligned
  configs via `MOLECULE_SCENARIO_DIRECTORY` so `dhcpd`/`dhcpd6` can start on EC2
  without changing production handlers.
- Task blocks use `rescue` for service/firewall errors while handlers remain
  strict — a follow-up issue may be warranted if bad configs should fail loudly
  at the handler stage in production.

## AAP

Uses the **Default execution environment** (in-repo role + `ansible.posix`).
Inventory group: `dhcp_servers`.
