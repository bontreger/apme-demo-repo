# playbook-linux-kdump-config scenario

Tests [`playbooks/linux-kdump-config.yml`](../../playbooks/linux-kdump-config.yml)
(which applies [`roles/kdump_config`](../../roles/kdump_config)): converge
configures kdump on `rhel9` with the default `Local` dump target, and
`verify.yml` asserts kexec-tools is installed, `/etc/kdump.conf` and the
panic-trigger sysctl file were written, and the kdump service is enabled.

## ⚠️ Run with awareness — this leaves the instance panic-prone

The role applies kernel **panic triggers live via sysctl** (`vm.panic_on_oom`,
`kernel.softlockup_panic`, `kernel.hung_task_panic`, `kernel.unknown_nmi_panic`
= 1). After converge, the instance will **kernel-panic (crash)** if any of those
conditions occur. `verify` is light so the practical risk is low, and the
instance is destroyed at the end — but a panic mid-run could orphan the EC2
instance, so run this deliberately, not as part of an unattended `--all`.

To exercise the config safely without the live panic triggers, override
`kdump_config_panic_triggers` (all keys `false`) via role params in a custom
converge; the default scenario runs the playbook faithfully.

## Notes on kdump behavior in this environment

- `kdumpctl reset-crashkernel` edits the kernel cmdline for the **next boot**;
  the role does not reboot, so kdump won't actually be armed until a reboot.
- `systemctl restart kdump` therefore fails on a non-rebooted instance (no
  crashkernel memory reserved) — handled by `failed_when: false`. verify checks
  the service is **enabled**, not necessarily active.

## Files

- `molecule.yml` — rhel9 only.
- `converge.yml` — imports the playbook under test.
- `inventory/hosts.yml` — maps `rhel9` into `all_rhel`.
- `verify.yml` — asserts kexec-tools, `/etc/kdump.conf`, the panic sysctl file,
  and kdump enabled.

## Running

```bash
molecule test -s playbook-linux-kdump-config
```

Cheap checks that need no AWS credentials:

```bash
molecule syntax -s playbook-linux-kdump-config
ansible-lint molecule/playbook-linux-kdump-config/
```
