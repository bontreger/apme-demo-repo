# ACL Ready-to-Use Content -- Specifications

Architectural and operational decisions governing the `acl-ready-to-use-content`
repository. This document is the single source of truth for contributors and
consumers of the content library.

---

## 1. Repository Classification

| Decision | Value | Rationale |
|----------|-------|-----------|
| ACL category | **Starter pack** | Content is basic, ready-to-use automation -- not multi-week engagements |
| Repository name | `acl-ready-to-use-content` | Follows ACL naming: `acl-<content-type>` |
| Excluded as ADUC | Scenarios 200 (VMware + OCP-V), 301 (Convert2RHEL) | Too complex for "ready-to-use"; each warrants its own ADUC-level repository |

## 2. Repository Layout

```
acl-ready-to-use-content/
├── playbooks/              # Flat .yml files -- no subdirectories
├── roles/                  # All roles hoisted to repo root
├── collections/            # requirements.yml for runtime collections
├── execution_environments/ # ansible-builder manifest
├── controller_config/      # AAP Configuration as Code
├── inventory/              # Example inventory + group_vars
├── docs/                   # Cross-cutting guides and this spec
└── ansible.cfg             # roles_path = roles
```

Key layout rules:

- **No subdirectories under `playbooks/`.**  Each playbook is a single flat
  `.yml` file named with a platform prefix.
- **All roles live under `roles/`** and are discovered via `ansible.cfg`
  (`roles_path = roles`).
- **Scenario-specific variables** go into `roles/<role>/defaults/main.yml`,
  not into `group_vars/` outside the role.
- **Role documentation** (`README.md`) lives inside each role directory.
- **Repository-wide settings** (e.g. reporting toggles) live in
  `inventory/group_vars/all/`.

## 3. Naming Conventions

| Element | Convention | Examples |
|---------|-----------|----------|
| Playbook files | `<platform>-<use-case>.yml` | `linux-dns-primary.yml`, `win-service-management.yml` |
| Platform prefixes | `linux-` for RHEL/Linux, `win-` for Windows | -- |
| Role directories | `snake_case`, no platform prefix unless Windows-only | `dns_sync`, `win_service_management` |
| AAP job template names | `ACL_<platform>_<use_case>` | `ACL_linux_dns_bind_primary`, `ACL_win_iis_deploy` |
| AAP credential names | `acl_<purpose>` | `acl_machine_credential`, `acl_windows_credential` |
| AAP inventory name | `acl_lab_inventory` (default, overridable) | -- |
| Role variables | `<rolename>_<variable>` prefix per GPA | `selinux_config_state`, `win_chocolatey_packages` |

## 4. Playbook Design

- **Thin entry points.** Playbooks contain minimal logic: host targeting,
  `become`, `gather_facts`, an optional RHEL version assert in `pre_tasks`,
  and a `roles:` list. All task logic lives in the role.
- **Static host groups.** Playbooks use hardcoded `hosts:` values
  (`all_rhel`, `all_windows`, or service-specific groups like `primary_dns`).
  Operators narrow scope at launch time using AAP's **limit** field
  (`ask_limit_on_launch: true` on all templates). No Jinja2 indirection in
  the `hosts:` directive.
- **No `import_playbook` across scenarios.** Each playbook is self-contained
  (may call shared utility roles like `report_publish`).

## 5. Role Design

- **Self-contained.** Each role includes `tasks/`, `defaults/main.yml`,
  `handlers/` (if needed), `templates/`, `files/`, and `README.md`.
- **Parameterized.** Where multiple legacy playbooks did the same thing with
  minor variations, they are consolidated into one role with a mode/action
  parameter (e.g. `win_service_action: start | stop | restart | check`).
- **GPA variable naming.** All role defaults are prefixed with the role name
  to avoid collisions (e.g. `firewall_management_zones`, not `zones`).
- **Scenario `group_vars` absorbed.** Any `group_vars/` from legacy scenarios
  are merged into `roles/<role>/defaults/main.yml` during migration.

## 6. Consolidation Decisions

| Family | Source scenarios | Result | Key parameter |
|--------|-----------------|--------|---------------|
| Web service RCA | 17 (httpd) + 18 (nginx) | 1 role `web_service_rca` | `web_service_rca_service: httpd \| nginx` |
| Memory RCA | 21 (high mem) + 22 (OOM) | 1 role `memory_rca` | `memory_rca_mode: high_utilization \| oom` |
| Diagnostic RCA | 29, 30, 31, 32 | 1 role `linux_diagnostic_rca` | `diagnostic_check_type: zombie_process \| disk_io \| systemd_unit \| privileged_account` |
| Yum repo config | 04 (3 playbooks) | 1 role `yum_repo_config` | `yum_repo_config_mode: server \| client \| deploy` |
| Win service mgmt | 81 (6 playbooks) | 1 role `win_service_management` | `win_service_action: check \| start \| stop \| restart` |
| Win software | 83 (2 playbooks) | 1 role `win_software_management` | `win_software_state: present \| absent` |
| Win Chocolatey | 84 (2 playbooks) | 1 role `win_chocolatey` | `win_chocolatey_state: present \| absent` |
| Win patching | 85 (3 playbooks) | 1 role `win_patch_management` | `win_patch_mode: install_all \| install_kb \| report` |

## 7. AAP Configuration as Code (CaC)

### Collection

| Collection | Purpose | Source |
|------------|---------|--------|
| `infra.aap_configuration` | CaC roles for AAP 2.5+ | Automation Hub |
| `ansible.controller` | Dependency of `infra.aap_configuration` | Automation Hub |

`infra.controller_configuration` is **not used** -- it is deprecated for
AAP 2.5+ and has incompatibilities with `ansible.controller` 4.x.

### What CaC manages

| AAP object | Managed by CaC | Notes |
|------------|:-:|-------|
| Inventory (shell) | Yes | Creates `acl_lab_inventory`; hosts added separately |
| Project | Yes | Idempotent -- updates if project already exists |
| Job templates (40) | Yes | 35 Linux + 5 Windows |
| Credentials | **No** | User must pre-create machine credentials in AAP |
| Hosts / groups | **No** | User adds hosts to inventory via AAP UI or API |

### User-provided variables

All site-specific values are provided via `controller_config/site_config.yml`
(copied from `site_config.yml.example`). Required variables:

| Variable | Required | Description |
|----------|:--------:|-------------|
| `aap_hostname` | Yes | AAP controller URL (e.g. `https://aap.example.com`) |
| `aap_validate_certs` | No | SSL verification; defaults to `true` |
| `aap_token` | Yes | Personal Access Token; passed at runtime via `-e` |
| `acl_project_scm_url` | Yes | Git URL of this repository |
| `acl_project_scm_branch` | No | Branch to track; defaults to `main` |
| `acl_project_scm_credential` | No | Name of SCM credential in AAP; omit for public repos |
| `acl_linux_credential` | Yes | Name of existing Machine credential for Linux hosts |
| `acl_windows_credential` | Yes | Name of existing Machine credential for Windows hosts |
| `acl_organization` | No | AAP organization; defaults to `Default` |
| `acl_inventory_name` | No | Inventory name; defaults to `acl_lab_inventory` |

### Prerequisites (manual)

Before running CaC the user must:

1. Create a Machine credential for Linux targets (e.g. `acl_machine_credential`).
2. Create a Machine credential for Windows targets (e.g. `acl_windows_credential`).
3. If the Git repository is private, create an SCM credential (SSH key or
   token) and reference its name in `acl_project_scm_credential`.
4. Optionally pre-create the project in AAP. CaC is idempotent -- if the
   project exists it will be updated, not duplicated.

## 8. Execution Environment

| Decision | Value |
|----------|-------|
| EE for job templates | `Default execution environment` |
| Custom EE | Available via `execution_environments/` manifest but not required |

The `Default execution environment` shipped with AAP 2.5 includes
`ansible.builtin`, `ansible.posix`, and `ansible.windows`. Additional
collections (e.g. `community.general`) must either be present in the EE or
installed into the project virtualenv. A custom EE build definition is
provided in `execution_environments/` for sites that need it.

## 9. Collections

Runtime collections required by playbooks are declared in
`collections/requirements.yml`:

- `ansible.posix`
- `ansible.windows`
- `community.general`
- `community.crypto`
- `community.mysql`
- `openstack.cloud`
- `community.windows`
- `chocolatey.chocolatey`

CaC collection (`infra.aap_configuration`) is listed separately -- it is only
needed on the workstation running the CaC playbook, not inside the EE.

## 10. Reporting Standard

Playbooks that produce diagnostic or audit reports follow the pattern defined
in `docs/reporting-standard.md`:

- Reports are generated into a local directory on the target host.
- The `report_publish` shared role optionally collects and publishes reports.
- Reporting is toggled via `report_publish_enabled` in
  `inventory/group_vars/all/reporting.yml` (default: `false`).

## 11. Content Scope

### Included (40 playbooks)

- **35 Linux playbooks**: DNS, DHCP, database, system config, inspection,
  diagnostics, RCA, security, network.
- **5 Windows playbooks**: service management, IIS, software, Chocolatey,
  patching.

### Excluded

| Scenario | Reason |
|----------|--------|
| 200 -- VMware + OCP-V | ADUC-level complexity |
| 301 -- Convert2RHEL | ADUC-level complexity |
| 101/102/103 -- VMware docs | Removed; belongs in a future ADUC VMware repo |
