## ACL Ready-to-Use Content

Ansible Content Library (ACL) — production-ready automation content maintained by the
Ansible Services Platform Team.

This repository follows the
[Ansible Content Library specification](https://docs.google.com/document/d/1eJmRoc5Hwz0jhGYHpQaU4VDHmCOy-d3kNyXjIXBnlq0)
and [Red Hat COP Good Practices for Ansible (GPA)](https://redhat-cop.github.io/automation-good-practices/).

---

## Repository Structure

```
acl-ready-to-use-content/
├── playbooks/          # One directory per scenario (e.g. dns-config/)
│   └── <slug>/
│       ├── site.yml
│       └── README.md
├── roles/              # All roles (shared and scenario-specific)
│   ├── report_publish/
│   └── <role_name>/
├── collections/
│   └── requirements.yml
├── execution_environments/
│   └── execution-environment.yml
├── controller_config/  # AAP as-code (infra.controller_configuration)
│   ├── projects.yml
│   └── job_templates.yml
├── inventory/
│   └── group_vars/all/reporting.yml
├── docs/               # Reference documentation and guides
└── .cursor/skills/     # AI assistant guidelines
```

## Principles

1. **AAP 2.6+** — all content tested on current supported AAP versions.
2. **Inventory and CaC live in AAP** — playbooks stay thin; operator config is in inventory or surveys.
3. **Minimal configuration** — sensible defaults; survey overrides for site-specific values.
4. **Supported RHEL targets only** — RHEL in a supportable state (including ELS/EUS).
5. **Report publishing standard** — see [docs/reporting-standard.md](docs/reporting-standard.md).

## Getting Started

```bash
# Install shared collections
ansible-galaxy collection install -r collections/requirements.yml

# Run a scenario
ansible-playbook playbooks/<slug>/site.yml -i <inventory>

# Lint
pip install -r requirements-dev.txt
pre-commit install
pre-commit run --all-files
```

## Support

This content is supported by the Ansible Services Platform Team at Red Hat.
