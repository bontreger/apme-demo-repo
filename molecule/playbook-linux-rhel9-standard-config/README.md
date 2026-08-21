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
not exist — an address unroutable from AWS, where the fetch blocks until SSH
times out and the role's rescue skips the whole hosts configuration.

The scenario therefore provisions a **second instance**, `hosts-template`, as
the template server. It sits in the `molecule` group so the lifecycle creates
it, but outside the play's `rhel9` pattern so the playbook never configures it
and its `/etc/hosts` stays fixed. That is the topology the role is written for:
a dedicated server whose hosts file is the source of truth.

`prepare.yml` writes a marker entry into that server's `/etc/hosts`, and
`verify.yml` asserts the marker arrived on the target — proving the
fetch/copy/merge chain end to end rather than just that the file exists.

Pointing `template_servers` at the target itself is cheaper (one instance) but
makes the template and the live file converge on each other, which hides
whether the merge is stable.

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

The RHEL 9 AMI ships without firewalld; `firewall_config` installs it, so the
port assertions exercise a real install-configure-open path. A failure there
means the role fell back to skipping the ports it was asked to open.

## Running

```bash
molecule test -s playbook-linux-rhel9-standard-config
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-rhel9-standard-config
ansible-lint molecule/playbook-linux-rhel9-standard-config/
```
