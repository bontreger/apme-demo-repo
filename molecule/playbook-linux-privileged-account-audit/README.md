# playbook-linux-privileged-account-audit scenario

Tests [`playbooks/linux-privileged-account-audit.yml`](../../playbooks/linux-privileged-account-audit.yml)
(which applies [`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)
with `check_type: privileged_account`): `prepare.yml` drops a NOPASSWD sudoers
entry, converge runs the audit, and `verify.yml` asserts the audit detected it
(WARNING) while reporting zero CRITICAL findings.

## What it exercises

The audit checks three things: non-root UID-0 users and empty-password accounts
(CRITICAL), and NOPASSWD sudoers entries (WARNING). `prepare.yml` creates only
the benign WARNING condition (a NOPASSWD entry for `molaudittest`), so:

- `sudo_nopasswd_entries.count >= 1` and the entry is found,
- `rogue_uid0_users.count == 0`, `empty_password_accounts.count == 0`,
- `audit_summary.security_status == 'WARNING'`.

`linux_diagnostic_rca_fail_on_critical` defaults to false, and no CRITICAL
condition is created, so the play never aborts.

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
- `prepare.yml` — drops a NOPASSWD sudoers entry for the audit to find.
- `verify.yml` — asserts both reports exist, the plaintext shape, and the JSON
  audit results (NOPASSWD detected, zero critical, status WARNING).

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped and no archive host is needed.

## Running

```bash
molecule test -s playbook-linux-privileged-account-audit
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-privileged-account-audit
ansible-lint molecule/playbook-linux-privileged-account-audit/
```
