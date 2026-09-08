# playbook-linux-ssl-cert-check scenario

Tests [`playbooks/linux-ssl-cert-check.yml`](../../playbooks/linux-ssl-cert-check.yml)
(`roles/ssl_cert_check`) on the molecule controller. No EC2 guests.

Requires outbound HTTPS (443) to the sites under test. The AAP job log is the
durable operator output; the local file is for CLI/Molecule.

## Files

- `molecule.yml` — localhost-only test sequence (no create/destroy)
- `converge.yml` — imports the playbook
- `verify.yml` — asserts `/tmp/ssl_cert_check-<hostname>.txt` lists both sites

## Running

```bash
molecule test -s playbook-linux-ssl-cert-check
```
