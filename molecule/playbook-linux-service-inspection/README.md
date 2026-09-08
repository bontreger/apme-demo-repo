# playbook-linux-service-inspection scenario

Tests [`playbooks/linux-service-inspection.yml`](../../playbooks/linux-service-inspection.yml)
([`roles/service_check`](../../roles/service_check)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts service status report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-service-inspection
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-service-inspection
ansible-lint molecule/playbook-linux-service-inspection/
```
