# playbook-linux-network-fault-diagnosis scenario

Tests [`playbooks/linux-network-fault-diagnosis.yml`](../../playbooks/linux-network-fault-diagnosis.yml)
([`roles/report_generation`](../../roles/report_generation)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts network diagnosis report on rhel9 and rhel10 (eth0, capture disabled).

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-network-fault-diagnosis
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-network-fault-diagnosis
ansible-lint molecule/playbook-linux-network-fault-diagnosis/
```
