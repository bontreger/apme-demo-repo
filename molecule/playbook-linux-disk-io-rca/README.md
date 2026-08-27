# playbook-linux-disk-io-rca scenario

Tests [`playbooks/linux-disk-io-rca.yml`](../../playbooks/linux-disk-io-rca.yml)
([`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts disk I/O RCA reports on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-disk-io-rca
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-disk-io-rca
ansible-lint molecule/playbook-linux-disk-io-rca/
```
