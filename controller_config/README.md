# Controller Configuration (Configuration as Code)

YAML data files for provisioning Ansible Automation Platform objects using the
`infra.aap_configuration` collection.

These files do **not** affect `ansible-playbook` runs from the CLI. They define
how operators launch migrated content from AAP (projects, job templates, etc.).

## Layout

| File | Purpose |
|------|---------|
| `site_config.yml.example` | Template for site-specific values (copy to `site_config.yml`) |
| `projects.yml` | Project definition (links to this Git repo) |
| `job_templates.yml` | 40 job templates (35 Linux, 5 Windows) |
| `inventories.yml` | Inventory definition (empty -- add hosts via AAP) |
| `configure_aap.yml` | Dispatch playbook that applies all config |

## Quick start

```bash
# One-time: install the CaC collection
ansible-galaxy collection install infra.aap_configuration

# Create your site config (gitignored, not committed):
cp controller_config/site_config.yml.example controller_config/site_config.yml
# Edit site_config.yml with your AAP hostname, repo URL, credential names...

# Push all config to AAP
ansible-playbook controller_config/configure_aap.yml \
  -e @controller_config/site_config.yml \
  -e aap_token=YOUR_PAT_HERE
```

## What gets created

1. **Inventory** -- name from `acl_inventory_name` (default: `acl_lab_inventory`)
2. **Project** -- `acl-ready-to-implement-content` pointing to your Git repo URL
3. **Job templates** -- 40 templates referencing your credentials and inventory

## Prerequisites (before running CaC)

- **Machine credentials** for Linux and Windows targets must already exist
  in AAP. Enter their names in `site_config.yml`.
- **SCM credential** (if the repo is private) must already exist in AAP.
- **Hosts** are added to the inventory after CaC runs (via AAP UI or API).

For detailed instructions see [docs/aap-deployment-guide.md](../docs/aap-deployment-guide.md).
