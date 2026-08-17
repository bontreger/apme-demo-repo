# playbook-linux-service-inspection scenario

Tests [`playbooks/linux-service-inspection.yml`](../../playbooks/linux-service-inspection.yml)
(which applies [`roles/service_check`](../../roles/service_check)): converge
runs the inspection against `rhel9`, and `verify.yml` asserts the status report
was written with a section for each of the ten critical services and that sshd
is reported RUNNING (proving the report reflects live `service_facts`).

## rhel9 only (and a defect)

Drops the shared linux layer and provisions only `rhel9`. The playbook asserts
`ansible_distribution_major_version in ["7", "8", "9"]`, so it aborts on
`rhel10`. Sibling playbooks accept 10 and the role has no RHEL 10
incompatibility, so the omission looks like a defect. **This scenario tests the
playbook's contract as written (RHEL ≤ 9).**

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9`, maps it into `all_rhel`, and pins
  `report_job_id: molecule` so converge and verify resolve the same report path.
- `verify.yml` — asserts the report exists, has the header, has a section for
  each checked service, and shows sshd RUNNING.

`report_publish_enabled` defaults to `false`, so the playbook's trailing
publish import is skipped and no archive host is needed.

## Running

```bash
molecule test -s playbook-linux-service-inspection
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-service-inspection
ansible-lint molecule/playbook-linux-service-inspection/
```
