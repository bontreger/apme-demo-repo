# playbook-linux-selinux-config scenario

Tests [`playbooks/linux-selinux-config.yml`](../../playbooks/linux-selinux-config.yml)
(which applies [`roles/selinux_config`](../../roles/selinux_config)): converge
runs the SELinux configuration against `rhel9`, and `verify.yml` asserts the
persistent config was set to `disabled` and the runtime mode was moved off
`Enforcing`.

## Safe to run under molecule

With the role defaults, this does **not** break molecule's SSH connection and
does **not** reboot:

- Runtime state is set to `permissive` (`setenforce 0` equivalent) — takes
  effect immediately, no reboot, and permissive blocks nothing.
- The persistent `SELINUX=disabled` is only written to `/etc/selinux/config`
  and takes effect on the *next* boot, which molecule never performs before
  destroying the instance.

The current SSH session therefore stays up throughout.

## rhel9 only (and a defect)

Drops the shared linux layer and provisions only `rhel9`. The playbook asserts
`ansible_distribution_major_version in ["7", "8", "9"]`, so it aborts on
`rhel10`. Sibling playbooks accept 10 and the role has no RHEL 10
incompatibility, so the omission looks like a defect. **This scenario tests the
playbook's contract as written (RHEL ≤ 9).**

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9` and maps it into `all_rhel`.
- `verify.yml` — asserts `/etc/selinux/config` has `SELINUX=disabled` and
  `getenforce` is not `Enforcing`. The runtime end-state is robust to the AMI's
  starting mode (enforcing → permissive; already permissive/disabled → unchanged).

## Running

```bash
molecule test -s playbook-linux-selinux-config
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-selinux-config
ansible-lint molecule/playbook-linux-selinux-config/
```
