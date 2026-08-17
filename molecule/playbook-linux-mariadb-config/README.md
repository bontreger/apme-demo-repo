# playbook-linux-mariadb-config scenario

Tests [`playbooks/linux-mariadb-config.yml`](../../playbooks/linux-mariadb-config.yml)
(which applies [`roles/mariadb_server`](../../roles/mariadb_server)): converge
deploys MariaDB on `rhel9`, and `verify.yml` asserts the service is running, the
application database + user exist, the `test` database was removed, and the
initial backup file was written.

## Safe to run

Guest-side deploy. The only network touch is firewalld adding the `mysql`
service (additive; SSH is unaffected). All parameters have role defaults
(root/user passwords, db name), so the playbook runs without a survey — the
default passwords are test-only placeholders the role ships.

## `failed_when`-free rescue → verify is the signal

Every step is wrapped in `block`/`rescue` that only debug-prints on failure, so
converge reports success even if (say) the package install failed. The
assertions in `verify.yml` are the real pass/fail check.

## Files

- `molecule.yml` — rhel9 only.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — maps `rhel9` into `all_rhel`.
- `verify.yml` — asserts mariadb running/enabled, `webapp_db` + `webapp_user`
  present, `test` db removed, and the `/root` backup file written.

## Running

```bash
molecule test -s playbook-linux-mariadb-config
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-mariadb-config
ansible-lint molecule/playbook-linux-mariadb-config/
```
