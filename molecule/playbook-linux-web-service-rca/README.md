# playbook-linux-web-service-rca scenario

Tests [`playbooks/linux-web-service-rca.yml`](../../playbooks/linux-web-service-rca.yml)
([`roles/web_service_rca`](../../roles/web_service_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts httpd RCA report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-web-service-rca
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-web-service-rca
ansible-lint molecule/playbook-linux-web-service-rca/
```
