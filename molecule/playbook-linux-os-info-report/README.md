# playbook-linux-os-info-report scenario

Tests [`playbooks/linux-os-info-report.yml`](../../playbooks/linux-os-info-report.yml)
(which applies [`roles/os_info_report`](../../roles/os_info_report)): converge
runs the report against `rhel9`, and `verify.yml` asserts the plaintext report
was written with the expected sections and real gathered facts.

## Files

- `molecule.yml` — rhel9 only.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — maps `rhel9` into `all_rhel`, pins
  `os_info_report_job_id: molecule` for a deterministic report path.
- `verify.yml` — asserts the report exists and contains the OS/Kernel/Network/
  Disk sections plus the `RedHat` distribution value.

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped and no archive host is needed. HTML generation is off
by default (`os_info_report_generate_html: false`), so only the plaintext
report is asserted.

## Running

```bash
molecule test -s playbook-linux-os-info-report
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-os-info-report
ansible-lint molecule/playbook-linux-os-info-report/
```
