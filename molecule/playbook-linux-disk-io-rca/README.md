# playbook-linux-disk-io-rca scenario

Tests [`playbooks/linux-disk-io-rca.yml`](../../playbooks/linux-disk-io-rca.yml)
(which applies [`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)
with `check_type: disk_io`): converge runs the RCA on a stock RHEL 9 instance
and `verify.yml` asserts the JSON and plaintext reports were written with the
expected disk IO investigation structure.

## rhel9 only

This scenario drops the shared linux layer and provisions only `rhel9`. The
playbook's `pre_tasks` assert `ansible_distribution_major_version in
["7", "8", "9"]`, so it aborts on `rhel10`. No `prepare.yml` is needed — a
stock AMI is sufficient for iostat/pidstat collection (the role degrades
gracefully when `sysstat` tools are absent).

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9`, maps it into `all_rhel`, and pins
  `linux_diagnostic_rca_report_job_id: molecule`.
- `verify.yml` — asserts both reports exist and contain expected disk IO RCA
  fields.

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped.

## AAP

Template: `ACL_linux_disk_io_rca`. Inventory group: `all_rhel`. Default EE.
No extra variables required for lab runs.

## Running

```bash
molecule test -s playbook-linux-disk-io-rca
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-disk-io-rca
ansible-lint molecule/playbook-linux-disk-io-rca/
```
