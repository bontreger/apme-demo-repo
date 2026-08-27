# playbook-linux-filesystem-check scenario

Tests [`playbooks/linux-filesystem-check.yml`](../../playbooks/linux-filesystem-check.yml)
([`roles/filesystem_check`](../../roles/filesystem_check)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts filesystem health report on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-filesystem-check
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-filesystem-check
ansible-lint molecule/playbook-linux-filesystem-check/
```
