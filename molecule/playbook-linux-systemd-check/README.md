# playbook-linux-systemd-check scenario

Tests [`playbooks/linux-systemd-check.yml`](../../playbooks/linux-systemd-check.yml)
([`roles/linux_diagnostic_rca`](../../roles/linux_diagnostic_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts systemd unit RCA reports on rhel9 and rhel10.

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-systemd-check
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-systemd-check
ansible-lint molecule/playbook-linux-systemd-check/
```
