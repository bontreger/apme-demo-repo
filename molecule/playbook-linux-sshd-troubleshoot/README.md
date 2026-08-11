# playbook-linux-sshd-troubleshoot scenario

Tests [`playbooks/linux-sshd-troubleshoot.yml`](../../playbooks/linux-sshd-troubleshoot.yml)
(which applies [`roles/sshd_troubleshoot`](../../roles/sshd_troubleshoot)):
converge runs the SSH troubleshooter against `rhel9`, and `verify.yml` asserts
the diagnostic reports were written, the `/etc/hosts` remediation was applied,
and sshd is running/enabled.

## Safe to run under molecule

Despite the "auto-remediation" name, the role does **not** restart or
reconfigure sshd, so it will not sever molecule's own SSH connection. The only
service task is `ensure started + enabled` (idempotent — a running sshd is not
restarted), and the only mutation is a backed-up `/etc/hosts` entry.

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
- `verify.yml` — globs the timestamped report dir, asserts the four reports
  (DNS, SSH status, client-fix guide, monitor script) exist and are non-empty,
  the monitor script is executable, `/etc/hosts` gained the target hostname,
  and sshd is running/enabled.

No `prepare.yml` is needed: the role tolerates a missing `nslookup`
(`failed_when: false` on the DNS tests), and the RHEL AMI already has sshd,
`ss`, and `ssh-keygen`.

## Running

```bash
molecule test -s playbook-linux-sshd-troubleshoot
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-sshd-troubleshoot
ansible-lint molecule/playbook-linux-sshd-troubleshoot/
```
