# playbook-linux-filesystem-check scenario

Tests [`playbooks/linux-filesystem-check.yml`](../../playbooks/linux-filesystem-check.yml)
and [`roles/filesystem_check`](../../roles/filesystem_check): converge runs
read-only filesystem health checks on a stock RHEL 9 instance and `verify.yml`
asserts the plaintext report was written.

## rhel9 only

Drops the shared linux layer and provisions only `rhel9`. The playbook's
`pre_tasks` assert `ansible_distribution_major_version in ["7", "8", "9"]`, so it
aborts on `rhel10`. No `prepare.yml` is needed — a stock AMI has mountable XFS
on `/boot` (root `/` is excluded by role defaults).

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9` in `molecule` and `all_rhel`; pins
  `report_job_id: molecule`.
- `verify.yml` — asserts the plaintext report exists with expected sections.

`report_publish_enabled` and `generate_html` default to `false`, so HTML and
publish plays are skipped.

## AAP

Template: `ACL_linux_filesystem_check`. Inventory group: `all_rhel`. Default EE.
No extra variables required.

## Running

```bash
molecule test -s playbook-linux-filesystem-check
```

```bash
molecule syntax -s playbook-linux-filesystem-check
ansible-lint molecule/playbook-linux-filesystem-check/
```
