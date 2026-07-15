# Controller configuration (Configuration as Code)

YAML data files for provisioning Ansible Automation Platform objects from this
repository using the `infra.controller_configuration` collection.

These files do **not** affect `ansible-playbook` runs from the CLI. They define
how operators launch migrated content from AAP (projects, job templates, etc.).

## Layout

| File | AAP objects |
|------|-------------|
| `projects.yml` | Projects (link to this Git repo) |
| `job_templates.yml` | Job templates for migrated pilot playbooks |

Grow incrementally: add `inventories.yml`, `credentials.yml`, and
`organizations.yml` when a target AAP instance is known.

## Placeholder values

Job templates reference **placeholder** organization, inventory, credential, and
execution environment names. Replace these with site-specific values before
applying to a controller:

- `organization` — your AAP organization
- `inventory` — lab or customer inventory configured in AAP
- `credentials` — machine credential for managed hosts
- `execution_environment` — EE built from `execution_environments/`

## Apply (example)

From a control host with `infra.controller_configuration` installed and AAP
credentials configured:

```bash
# Dispatch pattern — adjust playbook and auth to your environment.
ansible-playbook configure_controller.yml \
  -e "{controller_configuration_projects_async: false}"
```

See [docs/getting-started.md](../docs/getting-started.md) and
[docs/architecture.md](../docs/architecture.md) for the repo layout and CaC model.
