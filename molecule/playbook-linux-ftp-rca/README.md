# playbook-linux-ftp-rca scenario

Tests [`playbooks/linux-ftp-rca.yml`](../../playbooks/linux-ftp-rca.yml) and
[`roles/ftp_rca`](../../roles/ftp_rca): converge runs FTP RCA on a stock RHEL 9
instance in `all_rhel` + `ftp_servers`, and `verify.yml` asserts the plaintext
report was written with server and journal sections.

## rhel9 only

This scenario drops the shared linux layer and provisions only `rhel9`. The
playbook's `pre_tasks` assert `ansible_distribution_major_version in
["7", "8", "9"]`, so it aborts on `rhel10`. No `prepare.yml` is needed — the
role reports vsftpd package/service state on a stock AMI (typically not
installed).

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9` in `molecule`, `all_rhel`, and
  `ftp_servers`; pins `ftp_rca_report_job_id: molecule`.
- `verify.yml` — asserts the diagnosis report exists and has expected sections.

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped.

## AAP

Template: `ACL_linux_ftp_rca`. Inventory group: `all_rhel`. Default EE. No
extra variables required for basic lab runs. Add the host to `ftp_servers` for
full server-side checks (optional).

## Running

```bash
molecule test -s playbook-linux-ftp-rca
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-ftp-rca
ansible-lint molecule/playbook-linux-ftp-rca/
```
