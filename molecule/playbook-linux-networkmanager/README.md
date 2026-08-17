# playbook-linux-networkmanager scenario

Tests [`roles/networkmanager_cli`](../../roles/networkmanager_cli) (the role
behind [`playbooks/linux-networkmanager.yml`](../../playbooks/linux-networkmanager.yml))
safely: it exercises the view / add / activate paths against an **isolated
NetworkManager dummy device**, never the real NIC.

## Why it applies the role directly (not the playbook)

The role's `vars/main.yml` ships **live example operations** (add `static-eth1`
/ `dhcp-eth2`, activate them, modify DNS, delete named connections). `vars/`
is role-vars precedence, so running the role with no override *executes* those
examples — and `control.yml`'s `nmcli con up` has no `failed_when`, so it
aborts the play when the example devices are absent. That makes a faithful
`molecule test` of the playbook both unsafe and failing.

The playbook's survey `set_fact` only overrides the add/activate lists (leaving
the example modify/delete ops live), so importing the playbook can't neutralise
the danger either. **Role params** (as used in `converge.yml`) outrank
`vars/main.yml`, so the converge can replace every list with a safe value.

## Safe by construction

- The connection is `type: dummy` bound to `dummy0` — NetworkManager creates
  and manages an isolated virtual device; the real interface is never touched.
- It has a static IP (`10.99.99.2/24`) and **no gateway**, so no default route
  changes. `modify`/`deactivate`/`delete` lists are emptied.
- `verify.yml` ends with an `ansible.builtin.ping` to confirm the host is still
  reachable (SSH/control path intact).

## Files

- `molecule.yml` — rhel9 only.
- `prepare.yml` — `modprobe dummy` so NM can create the dummy device.
- `converge.yml` — applies `networkmanager_cli` with safe dummy-only role params.
- `verify.yml` — asserts the connection exists, is `activated`, `dummy0` has the
  static IP, and the host is still reachable.

## Findings recorded during review (not fixed here)

1. `vars/main.yml` ships dangerous active defaults (should be in `defaults/` or docs).
2. Survey `set_fact` only partially overrides those (modify/delete still run).
3. `control.yml` activation has no `failed_when`.
4. Playbook OS gate omits RHEL 10; bare `ansible_*` deprecations in the playbook.

## Running

```bash
molecule test -s playbook-linux-networkmanager
```
