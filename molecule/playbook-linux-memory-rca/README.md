# playbook-linux-memory-rca scenario

Tests [`playbooks/linux-memory-rca.yml`](../../playbooks/linux-memory-rca.yml)
([`roles/memory_rca`](../../roles/memory_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts memory RCA report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-memory-rca
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-memory-rca
ansible-lint molecule/playbook-linux-memory-rca/
```
