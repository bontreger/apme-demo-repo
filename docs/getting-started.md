# Getting Started

Quick start for running ACL ready-to-use content from the command line or
Ansible Automation Platform (AAP).

## Prerequisites

- Ansible 2.14+ (or current AAP-supported ansible-core)
- SSH access to lab hosts
- Supported targets: RHEL 7/8/9/10 in a supportable state (scenario-specific;
  see each playbook README)

## Clone and install collections

```bash
git clone https://github.com/redhat-services-platform-team/acl-ready-to-use-content.git
cd acl-ready-to-use-content

# Shared collections (repo root)
ansible-galaxy collection install -r collections/requirements.yml

# DHCP client playbook only — not in collections/requirements.yml.
# Requires Automation Hub (or RHEL host with rhel-system-roles RPM):
ansible-galaxy collection install redhat.rhel_system_roles
```

Collections are also listed in `execution_environments/requirements.yml` for EE
builds (includes `redhat.rhel_system_roles`). Build the custom EE before running
`ACL_linux_dhcp_client` in AAP — the platform Default EE does not include this
collection.

## Inventory

Copy and edit the example inventory:

```bash
cp inventory/inventory.example inventory/lab.ini
```

Populate host groups for the scenario you are testing. Common groups:

| Group | Used by |
|-------|---------|
| `primary_dns` | BIND primary DNS |
| `secondary_dns` | BIND secondary DNS |
| `caching_dns` | Unbound caching DNS |
| `dhcp_servers` | DHCP server |
| `dhcp_clients` | DHCP client |
| `all_rhel` | RHEL9 standard config, network fault diagnosis |

Diagnostic playbooks write plaintext under `/var/lib/ansible-reports/` on
each managed host and print the same content in the job log. See
[reporting-standard.md](reporting-standard.md).

## Run playbooks

**Always run from the repository root** so `ansible.cfg` resolves
`roles_path = roles`.

```bash
# Example: MariaDB
ansible-playbook playbooks/linux-mariadb-config.yml -i inventory/lab.ini

# Example: BIND primary DNS
ansible-playbook playbooks/linux-dns-primary.yml -i inventory/lab.ini
```

All playbooks are flat `.yml` files directly under `playbooks/` with a
`linux-` or `win-` prefix. Each playbook's corresponding role contains a
`README.md` with variables, limitations, and troubleshooting.

## Execution environment (optional)

Build a custom EE when using AAP:

```bash
ansible-builder build \
  -f execution_environments/execution-environment.yml \
  -t acl-ready-to-use-content-ee
```

## AAP deployment (optional)

CaC definitions for AAP live in `controller_config/`. For full deployment
instructions see [docs/aap-deployment-guide.md](aap-deployment-guide.md).

## Lint

```bash
pip install -r requirements-dev.txt
pre-commit install
pre-commit run --all-files
```

## Further reading

- [architecture.md](architecture.md) -- repo layout and migration model
- [specifications.md](specifications.md) -- all architectural decisions
- [aap-deployment-guide.md](aap-deployment-guide.md) -- AAP CaC deployment
- [reporting-standard.md](reporting-standard.md) -- report file and job-log pattern
- Per-role READMEs under `roles/<name>/README.md`
