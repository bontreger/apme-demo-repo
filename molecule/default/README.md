# default scenario

Smoke test for the molecule test infrastructure itself: provisions `rhel10`,
`rhel9`, and `win2022` (mixed SSH/WinRM), waits for connectivity, gathers
facts, and asserts each instance is the OS it is supposed to be. Use it to
validate provisioner changes (`molecule/utils/`), credentials, and platform
prerequisites before writing content scenarios.

## Files

- `molecule.yml` — restates the inventory stack with the windows layer added
  (the Windows opt-in pattern from the main README); everything else is
  inherited from `.config/molecule/config.yml`.
- `converge.yml` — prints a hello with the gathered distribution/version per
  host.
- `verify.yml` — asserts `ansible_os_family` matches each host's
  `molecule_expected_os_family`, and `ansible_distribution_major_version`
  matches `molecule_expected_major` where defined.
- `inventory/hosts.yml` — scenario-local overlay, layered after the shared
  inventories (later sources win). It adds the
  `molecule_expected_*` values to the shared hosts — they are only meaningful
  to this scenario's verify, which is why they live here and not in the shared
  `molecule/utils/inventory/`.

## Running

```bash
molecule test
```

Platform prerequisites and per-host image/connection selection are documented
in [molecule/README.md](../README.md).
