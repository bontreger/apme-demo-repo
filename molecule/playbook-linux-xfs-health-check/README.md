# playbook-linux-xfs-health-check scenario

Tests [`playbooks/linux-xfs-health-check.yml`](../../playbooks/linux-xfs-health-check.yml)
([`roles/xfs_health_check`](../../roles/xfs_health_check)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts XFS health report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-xfs-health-check
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-xfs-health-check
ansible-lint molecule/playbook-linux-xfs-health-check/
```
