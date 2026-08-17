# playbook-linux-pcp-analysis scenario

Tests [`playbooks/linux-pcp-analysis.yml`](../../playbooks/linux-pcp-analysis.yml)
(three plays: install on `all_rhel`, collect on `pcp_servers`, integrated report
on `localhost` via `roles/pcp_analysis`).

## rhel9 only

Provisions `rhel9` in `all_rhel` and `pcp_servers`. Play 3 delegates report
generation to `report_archive_host`; this scenario sets that to `rhel9` (the
molecule guest — same pattern as AAP with `report_archive_host: rhel9_lab`).

## Files

- `inventory/hosts.yml` — `rhel9` in `molecule`, `all_rhel`, and `pcp_servers`
- `inventory/group_vars/all.yml` — `report_archive_host: rhel9`
- `verify.yml` — asserts PCP is installed and an integrated
  `integrated_pcp_report_*.md` exists under `/var/www/ansible-reports/os_pcp/`
  on the guest

Expect a longer converge (PCP package install + collection).

## AAP

Template: `ACL_linux_pcp_analysis`. Put the lab host in **`all_rhel`** and
**`pcp_servers`**. Set `report_archive_host` if using the report play against
a real archive server; otherwise confirm play 3 behaviour with your team
(report play delegates to archive host even when publish is disabled).

## Running

```bash
molecule test -s playbook-linux-pcp-analysis
```
