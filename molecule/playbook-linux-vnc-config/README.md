# playbook-linux-vnc-config scenario

Tests [`playbooks/linux-vnc-config.yml`](../../playbooks/linux-vnc-config.yml)
(which applies [`roles/vnc_config`](../../roles/vnc_config)): converge runs the
role against `rhel9`, and `verify.yml` asserts the role's deterministic
outputs — the VNC user, the installed server package, the password file, and
the three config files.

## Trimmed test

The role default `vnc_config_packages` is `["tigervnc-server", "@workstation"]`
— `@workstation` is the full GNOME environment (~1 GB, several minutes to
install). This scenario overrides it to `tigervnc-server` only (in
`inventory/hosts.yml`) so the run stays fast and exercises the role's
user/config/password logic. It therefore does **not** install a desktop or
start the `vncserver@:2` service (which needs the GNOME session). Switch the
package list back to the role default and add a service assertion if you want
the full end-to-end path.

It also supplies a test password (`vnc_config_password: vncpass1`): the role
default is an empty string, which fails the role's own `>= 6` character check —
except every task in the role has `failed_when: false`, so that failure is
silently swallowed.

## `failed_when: false` everywhere → verify is the only signal

All 21 tasks in the role set `failed_when: false`, so **converge reports
success no matter what actually happens on the host.** The assertions in
`verify.yml` are the only real pass/fail signal, and are intentionally strict.
This is worth flagging in the peer review (see below).

## rhel9 only (and a defect)

Drops the shared linux layer and provisions only `rhel9`. The playbook asserts
`ansible_distribution_major_version in ["7", "8", "9"]`, so it aborts on
`rhel10`. Sibling playbooks accept 10, and the role has no RHEL 10
incompatibility, so the omission looks like a defect. **This scenario tests the
playbook's contract as written (RHEL ≤ 9).**

## Files

- `molecule.yml` — restates the inventory stack without the linux layer.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — declares `rhel9`, maps it into `all_rhel`, and sets
  the trimmed package list + test password.
- `verify.yml` — asserts user, package, password file, and the three config
  files.

## Running

```bash
molecule test -s playbook-linux-vnc-config
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-vnc-config
ansible-lint molecule/playbook-linux-vnc-config/
```
