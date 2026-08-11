# playbook-linux-zombie-process-rca scenario

Tests [`playbooks/linux-zombie-process-rca.yml`](../../playbooks/linux-zombie-process-rca.yml)
(which applies [`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)
with `check_type: zombie_process`): `prepare.yml` manufactures a real zombie
process, converge runs the RCA, and `verify.yml` asserts the JSON and plaintext
reports were written and that the zombie was actually detected.

## rhel9 only (and a defect)

Unlike the sibling RCA scenarios, this one drops the shared linux layer and
provisions only `rhel9`. The playbook's `pre_tasks` assert
`ansible_distribution_major_version in ["7", "8", "9"]`, so it aborts on
`rhel10` with "Unsupported operating system." The sibling RCA playbooks accept
10 (`linux-chronyd-rca` → 7/8/9/10, `linux-web-service-rca` → 8/9/10), and the
role's tasks have no RHEL 10 incompatibility, so the omission looks like a
defect rather than an intentional limit.

**This scenario tests the playbook's contract as written (RHEL ≤ 9).** If the
gate is fixed to include "10", add `rhel10` back here (map it into `all_rhel`
and into `molecule`, or simply revert to the empty-`molecule.yml` +
both-hosts pattern the sibling scenarios use).

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9`, maps it into `all_rhel`, and pins
  `linux_diagnostic_rca_report_job_id: molecule` so converge and verify resolve
  the same report path (the default job id is `ansible_date_time.epoch`, which
  changes between plays).
- `prepare.yml` — spawns a transient systemd unit whose process forks a child
  and never reaps it, leaving a genuine zombie for the RCA to find.
- `verify.yml` — asserts both reports exist, the plaintext has the expected
  sections, and the parsed JSON shows `zombie_count >= 1` with matching detail.

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped and no archive host is needed.

## Running

```bash
molecule test -s playbook-linux-zombie-process-rca
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-zombie-process-rca
ansible-lint molecule/playbook-linux-zombie-process-rca/
```
