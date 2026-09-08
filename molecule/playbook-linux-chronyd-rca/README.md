# playbook-linux-chronyd-rca scenario

Tests [`playbooks/linux-chronyd-rca.yml`](../../playbooks/linux-chronyd-rca.yml)
([`roles/chronyd_rca`](../../roles/chronyd_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts chronyd RCA report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-chronyd-rca
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-chronyd-rca
ansible-lint molecule/playbook-linux-chronyd-rca/
```
