# Getting Started

Quick start for running ACL ready-to-use content from the command line or
Ansible Automation Platform (AAP).

## Prerequisites

- Ansible 2.14+ (or current AAP-supported ansible-core)
- SSH access to lab hosts
- Supported targets: RHEL 7/8/9 in a supportable state (scenario-specific;
  see each playbook README)

## Clone and install collections

```bash
git clone https://github.com/redhat-services-platform-team/acl-ready-to-use-content.git
cd acl-ready-to-use-content

# Shared collections (repo root)
ansible-galaxy collection install -r collections/requirements.yml

# DHCP client playbook only — not in collections/requirements.yml
ansible-galaxy collection install rhel-system-roles.network
```

Collections are also listed in `execution_environments/requirements.yml` for EE
builds (includes `rhel-system-roles.network`).

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

Reporting toggles live in `inventory/group_vars/all/reporting.yml`
(`report_publish_enabled: false` by default).

## Run playbooks

**Always run from the repository root** so `ansible.cfg` resolves
`roles_path = roles`.

```bash
# Example: MariaDB
ansible-playbook playbooks/mariadb-config/deploy_mariadb_site.yml -i inventory/lab.ini

# Example: BIND primary DNS
ansible-playbook playbooks/dns-config/deploy_bind_primary_site.yml -i inventory/lab.ini
```

## Pilot batch (lab validation)

Migrated playbooks available for testing:

| Scenario | Playbook path |
|----------|---------------|
| DNS — BIND primary | `playbooks/dns-config/deploy_bind_primary_site.yml` |
| DNS — BIND secondary | `playbooks/dns-config/deploy_bind_secondary_site.yml` |
| DNS — Unbound | `playbooks/dns-config/deploy_unbound_site.yml` |
| DHCP — server | `playbooks/dhcp-config/deploy_dhcp_server_site.yml` |
| DHCP — client | `playbooks/dhcp-config/configure_dhcp_client_site.yml` |
| MariaDB | `playbooks/mariadb-config/deploy_mariadb_site.yml` |
| RHEL 9 standard config | `playbooks/rhel9-standard-config/rhel9_standardization_site.yml` |
| Network fault diagnosis | `playbooks/network-fault-diagnosis/network_diagnosis_site.yml` |
| OpenStack VM lifecycle | `playbooks/openstack-vm-lifecycle/site.yml` |

Each directory under `playbooks/<slug>/` has a README with variables,
limitations, and troubleshooting.

## Execution environment (optional)

Build a custom EE when using AAP:

```bash
ansible-builder build \
  -f execution_environments/execution-environment.yml \
  -t acl-ready-to-use-content-ee
```

## AAP job templates (optional)

CaC definitions for AAP live in `controller_config/`. See
[controller_config/README.md](../controller_config/README.md). Replace
placeholder inventory, credential, and organization names before applying to
a controller.

## Lint

```bash
pip install -r requirements-dev.txt
pre-commit install
pre-commit run --all-files
```

## Further reading

- [architecture.md](architecture.md) — repo layout and migration model
- [reporting-standard.md](reporting-standard.md) — report publishing pattern
- Per-scenario READMEs under `playbooks/<slug>/README.md`
