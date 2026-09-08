# playbook-linux-kernel-log-check scenario

Tests [`playbooks/linux-kernel-log-check.yml`](../../playbooks/linux-kernel-log-check.yml)
([`roles/kernel_log_monitor`](../../roles/kernel_log_monitor)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts kernel log report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-kernel-log-check
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-kernel-log-check
ansible-lint molecule/playbook-linux-kernel-log-check/
```
