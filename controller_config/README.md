# Controller configuration (Configuration as Code)

YAML data files for provisioning Ansible Automation Platform objects from this
repository using the `infra.controller_configuration` collection.

These files do **not** affect `ansible-playbook` runs from the CLI. They define
how operators launch migrated content from AAP (projects, job templates, etc.).

## Layout

| File | AAP objects |
|------|-------------|
| `projects.yml` | Projects (link to this Git repo) |
| `job_templates.yml` | Job templates for all 40 playbooks |
| `inventories.yml` | Inventory, hosts, and groups |
| `credentials.yml` | Credential renames to match job template references |
| `configure_aap.yml` | Dispatch playbook that applies all config |
| `controller_auth.yml` | **Local only** (gitignored) — AAP hostname |

## Quick start

```bash
# One-time: install the collection
ansible-galaxy collection install infra.controller_configuration

# Create controller_auth.yml (gitignored, not committed):
cat > controller_config/controller_auth.yml <<EOF
---
controller_hostname: https://your-aap-host.example.com
controller_validate_certs: false
EOF

# Push all config to AAP
ansible-playbook controller_config/configure_aap.yml \
  -e @controller_config/controller_auth.yml \
  -e controller_oauthtoken=YOUR_PAT_HERE
```

## What gets created

1. **Credentials** — renames existing credentials to `acl_machine_credential` / `acl_windows_credential`
2. **Inventory** — `acl_lab_inventory` with hosts and groups (`all_rhel`, `all_windows`)
3. **Project** — `acl-ready-to-use-content` pointing to this Git repo
4. **Job templates** — 40 templates (35 Linux, 5 Windows)
