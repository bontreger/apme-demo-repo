# Ansible Reporting Standard

This document defines the repo-wide pattern for diagnostic and audit reports.

## Delivery

1. Write plaintext on the **managed host** at
   `/var/lib/ansible-reports/<rolename>-<hostname>.txt` (overwrite each run).
2. Print the same content with `ansible.builtin.debug` so the **AAP job log**
   is the primary operator delivery mechanism.
3. Use **role-prefixed** variables only. Do not share unprefixed `report_*`
   globals.

On AAP, `delegate_to: localhost` and writes under the execution environment
are ephemeral. Files persist only on real managed hosts. Playbooks that run
on `localhost` (for example SSL certificate checks) still `debug` the report;
any local file is a convenience for CLI runs and Molecule, not an archive.

This starter pack does **not** publish reports to a central archive host.
Operators who need a central store can `fetch` files from
`/var/lib/ansible-reports/` in their own follow-on automation.

## Role variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `{rolename}_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host |
| `{rolename}_report_path` | `{{ {rolename}_report_dir }}/{rolename}-{{ inventory_hostname }}.txt` | Plaintext report (overwritten) |

Multi-mode roles may include the mode in the filename so concurrent use cases
do not clobber each other (for example
`linux_diagnostic_rca-<check_type>-<hostname>.txt`). Optional JSON uses the
same stem with a `.json` suffix.

## Playbook structure

Reporting playbooks are a single play: host targeting, `become`, facts, an
OS assert, and a `roles:` list. Report write and `debug` live **in the role**.
Do not import a second publish playbook. Do not load a shared reporting
`vars_files`.

```yaml
---
- name: Memory RCA
  hosts: all_rhel
  become: true
  gather_facts: true
  pre_tasks:
    - name: Validate OS compatibility
      ansible.builtin.assert:
        that:
          - ansible_os_family == "RedHat"
          - ansible_distribution_major_version in ["7", "8", "9", "10"]
        fail_msg: "Unsupported operating system."
      tags: [always]
  roles:
    - role: memory_rca
```

## Local testing

```bash
ansible-playbook playbook.yml -i inventory
# Report path on each managed host:
#   /var/lib/ansible-reports/<rolename>-<inventory_hostname>.txt
```
