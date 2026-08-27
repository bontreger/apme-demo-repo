# playbook-linux-os-info-report scenario

Tests [`playbooks/linux-os-info-report.yml`](../../playbooks/linux-os-info-report.yml)
([`roles/os_info_report`](../../roles/os_info_report)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts plaintext OS report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-os-info-report
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-os-info-report
ansible-lint molecule/playbook-linux-os-info-report/
```
