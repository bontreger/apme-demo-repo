# playbook-linux-ftp-rca scenario

Tests [`playbooks/linux-ftp-rca.yml`](../../playbooks/linux-ftp-rca.yml)
([`roles/ftp_rca`](../../roles/ftp_rca)). Converge runs against **rhel9 and rhel10**;
`verify.yml` asserts FTP RCA report on rhel9 and rhel10 (also in ftp_servers).

Reports land at `/var/lib/ansible-reports/<role>-<hostname>.txt` on each guest.

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-ftp-rca
```

Cheap checks:

```bash
molecule syntax -s playbook-linux-ftp-rca
ansible-lint molecule/playbook-linux-ftp-rca/
```
