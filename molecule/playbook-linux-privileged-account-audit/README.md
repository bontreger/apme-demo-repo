# playbook-linux-privileged-account-audit scenario

Tests [`playbooks/linux-privileged-account-audit.yml`](../../playbooks/linux-privileged-account-audit.yml)
([`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts privileged account audit reports on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-privileged-account-audit
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-privileged-account-audit
ansible-lint molecule/playbook-linux-privileged-account-audit/
```
