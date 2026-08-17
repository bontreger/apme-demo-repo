# playbook-linux-firewall-management scenario

Tests [`playbooks/linux-firewall-management.yml`](../../playbooks/linux-firewall-management.yml)
(which applies [`roles/firewall_management`](../../roles/firewall_management)):
converge runs the playbook with the `start` action against `rhel9`, and
`verify.yml` asserts firewalld ends up running and enabled.

## Action override

`firewall_management_action` defaults to `check` (read-only) for safety. Since
it is a role **default** (lowest precedence), `inventory/hosts.yml` overrides it
to `start` — this exercises the substantive install/start/enable path *through
the playbook* rather than just the status report.

## Connection-safe

Starting firewalld does not drop molecule's SSH:
- firewalld's default zone permits the `ssh` service (port 22),
- the already-established SSH connection survives the service start via
  connection tracking, and
- the AWS security group guards port 22 independently.

`verify.yml` also queries `firewall-cmd --query-service=ssh` and ends with a
reachability check implied by the tasks completing.

## Files

- `molecule.yml` — rhel9 only.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — maps `rhel9` into `all_rhel`, sets
  `firewall_management_action: start`.
- `verify.yml` — asserts firewalld running + enabled (service_facts +
  `firewall-cmd --state`), and reports the ssh-service allow status.

## Running

```bash
molecule test -s playbook-linux-firewall-management
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-firewall-management
ansible-lint molecule/playbook-linux-firewall-management/
```
