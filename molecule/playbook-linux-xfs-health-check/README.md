# playbook-linux-xfs-health-check scenario

Tests [`playbooks/linux-xfs-health-check.yml`](../../playbooks/linux-xfs-health-check.yml)
(which applies [`roles/xfs_health_check`](../../roles/xfs_health_check)):
converge runs the health check against `rhel9`, whose root filesystem is XFS,
and `verify.yml` asserts the plaintext report was written with all its expected
sections.

## rhel9 only (and a defect)

This scenario drops the shared linux layer and provisions only `rhel9`. The
playbook's `pre_tasks` assert `ansible_distribution_major_version in
["7", "8", "9"]`, so it aborts on `rhel10` with "Unsupported operating system."
The sibling RCA playbooks accept 10 (`linux-chronyd-rca` → 7/8/9/10,
`linux-web-service-rca` → 8/9/10), and the role's tasks have no RHEL 10
incompatibility, so the omission looks like a defect rather than an intentional
limit.

**This scenario tests the playbook's contract as written (RHEL ≤ 9).** If the
gate is fixed to include "10", add `rhel10` back here.

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9`, maps it into `all_rhel`, and pins
  `xfs_health_check_job_id: molecule` so converge and verify resolve the same
  report path (the default job id is `ansible_date_time.epoch`, which changes
  between plays).
- `verify.yml` — asserts the report exists and contains the header, kernel-log,
  per-filesystem, and footer sections.

No `prepare.yml` is needed: the RHEL root filesystem is XFS, so the role has a
real filesystem to inspect. `report_publish_enabled` defaults to `false`, so
the playbook's trailing publish import is skipped and no archive host is needed.

## Running

```bash
molecule test -s playbook-linux-xfs-health-check
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-xfs-health-check
ansible-lint molecule/playbook-linux-xfs-health-check/
```
