# playbook-linux-zombie-process-rca scenario

Tests [`playbooks/linux-zombie-process-rca.yml`](../../playbooks/linux-zombie-process-rca.yml)
([`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts zombie process RCA reports on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-zombie-process-rca
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-zombie-process-rca
ansible-lint molecule/playbook-linux-zombie-process-rca/
```
