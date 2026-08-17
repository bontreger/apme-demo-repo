# playbook-linux-memory-rca scenario

Tests [`playbooks/linux-memory-rca.yml`](../../playbooks/linux-memory-rca.yml) and
[`roles/memory_rca`](../../roles/memory_rca) in default `high_utilization` mode:
converge runs memory RCA on a stock RHEL 9 instance and `verify.yml` asserts the
plaintext report was written with expected sections.

## rhel9 only

This scenario drops the shared linux layer and provisions only `rhel9`. The
playbook's `pre_tasks` assert `ansible_distribution_major_version in
["7", "8", "9"]`, so it aborts on `rhel10`. No `prepare.yml` is needed.

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9`, maps it into `all_rhel`, pins
  `memory_rca_report_job_id: molecule` and `memory_rca_mode: high_utilization`.
- `verify.yml` — asserts the diagnosis report exists and has expected sections.

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped.

## AAP

Template: `ACL_linux_memory_rca`. Inventory group: `all_rhel`. Default EE.
Optional extra var: `memory_rca_mode: high_utilization` (default) or `oom`.

## Running

```bash
molecule test -s playbook-linux-memory-rca
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-memory-rca
ansible-lint molecule/playbook-linux-memory-rca/
```
