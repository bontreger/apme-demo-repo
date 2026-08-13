# playbook-linux-systemd-check scenario

Tests [`playbooks/linux-systemd-check.yml`](../../playbooks/linux-systemd-check.yml)
(which applies [`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)
with `check_type: systemd_unit`): `prepare.yml` manufactures a failed systemd
unit, converge runs the check, and `verify.yml` asserts the reports were
written and that the failed unit was actually detected.

## What this scenario caught

The role built its failed-units list with
`ansible_facts.services | selectattr('value.state', 'equalto', 'failed')`.
But `ansible.builtin.service_facts` records a systemd unit's `state` as only
`running`/`stopped` — a failed unit's `failed` value goes under `status`, not
`state`. So the filter never matched and the check always reported zero failed
units. The fix filters on `value.status`; this scenario's `prepare.yml` +
`verify.yml` are the regression test (verify fails on the old code, passes on
the fixed code).

## rhel9 only (and a defect)

Drops the shared linux layer and provisions only `rhel9`. The playbook asserts
`ansible_distribution_major_version in ["7", "8", "9"]`, so it aborts on
`rhel10`. Sibling RCA playbooks accept 10 and the role has no RHEL 10
incompatibility, so the omission looks like a defect. **This scenario tests the
playbook's contract as written (RHEL ≤ 9).**

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9`, maps it into `all_rhel`, and pins
  `linux_diagnostic_rca_report_job_id: molecule`.
- `prepare.yml` — installs a oneshot unit that runs `/bin/false`, leaving it in
  the failed state.
- `verify.yml` — asserts both reports exist, the plaintext has the expected
  sections, and the parsed JSON shows the failed unit was detected.

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped and no archive host is needed.

## Running

```bash
molecule test -s playbook-linux-systemd-check
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-systemd-check
ansible-lint molecule/playbook-linux-systemd-check/
```
