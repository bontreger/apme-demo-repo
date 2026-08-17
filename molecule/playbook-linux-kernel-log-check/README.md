# playbook-linux-kernel-log-check scenario

Tests [`playbooks/linux-kernel-log-check.yml`](../../playbooks/linux-kernel-log-check.yml)
(which applies [`roles/kernel_log_monitor`](../../roles/kernel_log_monitor)):
converge captures and classifies recent kernel logs on `rhel9` and writes a
report, and `verify.yml` asserts the report was written with its expected
sections.

A clean instance normally has no critical/severe kernel errors, so the report
records zero findings — verify checks the report's structure (header, the
Critical/Severe/Unique counts, and the sample sections), not a specific count,
so it's robust to whatever the kernel ring buffer contains.

## Files

- `molecule.yml` — rhel9 only.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — maps `rhel9` into `all_rhel`, pins
  `report_job_id: molecule` for a deterministic report path.
- `verify.yml` — asserts the report exists and has the header + count lines +
  Critical/Severe sample sections.

`report_publish_enabled` defaults to `false`, so the playbook's trailing publish
import is skipped; HTML generation is off by default (`generate_html: false`),
so only the plaintext report is asserted.

## Running

```bash
molecule test -s playbook-linux-kernel-log-check
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-kernel-log-check
ansible-lint molecule/playbook-linux-kernel-log-check/
```
