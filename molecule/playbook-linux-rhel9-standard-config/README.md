# playbook-linux-rhel9-standard-config scenario

Tests [`playbooks/linux-rhel9-standard-config.yml`](../../playbooks/linux-rhel9-standard-config.yml),
which applies seven roles in order: [`user_management`](../../roles/user_management),
[`system_config`](../../roles/system_config), [`firewall_config`](../../roles/firewall_config),
[`hosts_config`](../../roles/hosts_config), [`chrony_config`](../../roles/chrony_config),
[`system_optimization`](../../roles/system_optimization) and
[`verification`](../../roles/verification).

## rhel9 only

The play targets `hosts: rhel9` and its pre_task asserts
`ansible_distribution_major_version == "9"`, so the scenario drops the shared
linux layer (which also ships `rhel10`, which the playbook would reject) and
declares the host locally.

No group named `rhel9` is created: the play's host pattern matches the *host*
of that name directly, and adding a group with the same name makes Ansible warn
`Found both group and host with same name`.

## template_servers

`hosts_config` fetches `/etc/hosts` from `groups['template_servers'][0]`,
falling back to `template_server.host` (`10.66.208.237`) when that group does
not exist. That address is unroutable from AWS, so without the group the fetch
blocks until SSH times out and the role's rescue skips the whole hosts
configuration. The scenario points `template_servers` at the test host itself,
so the fetch/copy/merge path executes instead of being skipped.

## Safe to run under molecule

- SELinux moves to `permissive` at runtime (no reboot) and `SELINUX=permissive`
  is written for the next boot, which molecule never performs.
- `firewall_config` only adds ports; the SSH and WinRM rules the provisioner
  relies on are untouched, and established connections survive a firewalld
  start.
- The play's final reboot task is gated on `srv_reboot_required`, which nothing
  sets here, so it is skipped.
- `chrony_config` pauses 10 seconds waiting for sync and points at
  `ntp.aliyun.com`; reachability is not asserted, only the configuration and
  the service state.

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9` and the `template_servers` group.
- `verify.yml` — asserts one outcome per role: the admin user and its
  passwordless sudo rule, the persistent and runtime SELinux state, every
  configured firewall port, the host's own entry in `/etc/hosts`, the NTP
  servers plus a running and enabled `chronyd`, the `limits.conf` entries, and
  the sysctl drop-in with its values applied at runtime.

A failure on the firewalld assertion means the AMI has no firewalld and the
playbook silently skipped every port — the role warns rather than failing, so
the assertion is what surfaces it.

## Running

```bash
molecule test -s playbook-linux-rhel9-standard-config
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-rhel9-standard-config
ansible-lint molecule/playbook-linux-rhel9-standard-config/
```
