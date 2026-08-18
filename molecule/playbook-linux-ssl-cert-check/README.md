# playbook-linux-ssl-cert-check scenario

Tests [`playbooks/linux-ssl-cert-check.yml`](../../playbooks/linux-ssl-cert-check.yml)
and `roles/ssl_cert_check` (SSL expiry check + Markdown report on `localhost`).

## localhost only — no EC2

This playbook runs on `localhost` with an RHEL 7/8/9 OS gate. The molecule
controller is Amazon Linux, so `converge.yml` exercises the role directly
instead of importing the playbook. No `create`/`destroy` step — fast run.

Requires outbound HTTPS (443) to the sites under test.

## Files

- `converge.yml` — runs `ssl_cert_check` on localhost; pins
  `ssl_cert_check_job_id: molecule` and `/tmp/ssl_report_molecule.md`
- `verify.yml` — asserts the Markdown report exists and lists both test sites

## AAP

Template: `ACL_linux_ssl_cert_check`. Runs on the execution node (`localhost`);
**no lab host or inventory groups required**. Optional extra vars:

```yaml
ssl_cert_check_sites:
  - www.redhat.com
  - github.com
```

Play 2 (archive publish) runs only when `report_publish_enabled: true`.

## Running

```bash
molecule test -s playbook-linux-ssl-cert-check
```
