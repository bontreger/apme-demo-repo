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
| Catalog scope | **Subset of the Use Case Maturity Catalog** | Mostly `maturity:l2-process-oriented`, `lifecycle:day1` and `lifecycle:day2`; other ACL repositories may cover day0, L3/L4, and ADUC-scale content |
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
| Role public variables | `{rolename}_*` prefix per GPA | `selinux_config_state`, `win_chocolatey_packages` |
| Role internal variables | `__{rolename}_*` (double underscore) per GPA | `__web_service_rca_profile`, `__memory_rca_diagnosis` |

## 4. Playbook Design

- **Thin entry points.** Playbooks contain minimal logic: host targeting,
  `become`, `gather_facts`, an optional RHEL version assert in `pre_tasks`,
  and a `roles:` list. All task logic lives in the role.
- **Static host groups.** Playbooks use hardcoded `hosts:` values
  (`all_rhel`, `all_windows`, or service-specific groups like `primary_dns`).
  Operators narrow scope at launch time using AAP's **limit** field
  (`ask_limit_on_launch: true` on all templates). No Jinja2 indirection in
  the `hosts:` directive.
- **No `import_playbook` across scenarios.** Each playbook is self-contained.

### Playbook file header (comment block)

Every file under `playbooks/` must begin with a **YAML comment header** placed
**before** the document start marker (`---`). The header is catalog metadata for
contributors, operators, and cross-repository discovery. It does **not** replace
role `README.md` files and is **not** the same as Ansible runtime `tags:` on
tasks (see below).

#### Use Case Maturity Catalog (classification model)

Playbook and role metadata is anchored to the **Automation Use Cases Maturity
Catalog** (lifecycle × maturity matrix). That catalog is **broader** than this
repository: other ACL repos may populate different pillars, lifecycle stages, and
maturity levels. Tags describe **where a use case sits in the program catalog**,
not everything this repo must implement today.

Each artifact carries **three required classification tags** (prefixed), plus
optional extras:

| Axis | Tag prefix | Required | Meaning |
|------|------------|:--------:|---------|
| **Pillar** | `pillar:` | Yes | Technology domain |
| **Lifecycle** | `lifecycle:` | Yes | Day 0 / Day 1 / Day 2 stage |
| **Maturity** | `maturity:` | Yes | Automation maturity level (L1–L4) |

Optional extras (same `# Tags:` line): `platform:`, `tech:`, `context:` — see
vocabulary below. **Phase 1** uses the matrix only; free-form keyword / tag
clouds are deferred to a later catalog phase.

**Required header fields**

| Field | Required | Notes |
|-------|:--------:|-------|
| Playbook title | Yes | Short human title (banner line) |
| `Description:` | Yes | 1–3 lines: purpose and notable behaviour |
| `Tags:` | Yes | Prefixed tags; must include exactly one `pillar:`, `lifecycle:`, `maturity:` |
| `Supported OS:` | When restricted | Include when a `pre_task` assert restricts OS; omit otherwise |
| `Author:` | Yes | `Ansible Services Platform Team, Red Hat` |
| `Version:` | Yes | Semantic content version; start at `1.0.0` |
| `Date:` | Yes | `YYYY-MM-DD` — header added or last meaningful content change |
| `License:` | Yes | `Apache-2.0` unless a bundled role specifies otherwise |

**Canonical template**

```yaml
# ============================================================================
# Playbook: <Human title>
# ============================================================================
#
# Description:
#   <What this playbook does; key roles or side effects in 1–3 lines.>
#
# Tags: pillar:<pillar>,lifecycle:<stage>,maturity:<level>,platform:<linux|windows>
#
# Supported OS:
#   <e.g. RHEL 7/8/9 — only when a pre_task assert restricts OS>
#
# Author: Ansible Services Platform Team, Red Hat
# Version: 1.0.0
# Date: YYYY-MM-DD
# License: Apache-2.0
#
# ============================================================================
---
```

**Examples**

`linux-dns-primary.yml`:

```yaml
# Tags: pillar:network,lifecycle:day1,maturity:l2-process-oriented,platform:linux,tech:bind
```

`linux-cve-patching.yml`:

```yaml
# Tags: pillar:os,lifecycle:day2,maturity:l2-process-oriented,platform:linux,tech:patching
```

**Controlled vocabulary**

Tags are lowercase, comma-separated **without** spaces. New values require a
spec update (PR).

| Prefix | Allowed values |
|--------|----------------|
| `pillar:` | `os`, `cloud`, `network`, `db`, `security`, `ops` |
| `lifecycle:` | `day0`, `day1`, `day2` |
| `maturity:` | `l1-task-focus`, `l2-process-oriented`, `l3-event-driven`, `l4-ai-driven` |
| `platform:` (optional) | `linux`, `windows` |
| `tech:` (optional) | `bind`, `unbound`, `dhcp`, `mariadb`, `postgresql`, `nfs`, `firewall`, `selinux`, `kdump`, `pcp`, `ssl`, `systemd`, `iis`, `chocolatey`, `patching`, … — add via spec PR |
| `context:` (optional) | `localhost`, `multi-play`, `rhel9-only` |

**Classification rules**

- Assign **one** `pillar:`, **one** `lifecycle:`, and **one** `maturity:` per
  playbook (primary classification). Use `Description:` for nuance (e.g. bundles
  that span day1 and day2 tasks).
- **This repository** content is expected to be mostly `maturity:l2-process-oriented` and
  `lifecycle:day1` or `lifecycle:day2`. Use `day0`, `l3`, or `l4` only when
  the use case genuinely belongs in that catalog cell.
- **Building blocks** (roles, thin playbooks) are tagged at their own maturity.
  Future **collections** that compose mixed blocks carry their own matrix tags;
  do not force a single maturity across every included role.
- **RCA / inspection** playbooks: primary pillar is usually `ops` unless the
  use case is clearly scoped to one domain (e.g. network fault → `pillar:network`).

**Header `Tags:` vs Ansible runtime tags**

| | Header `# Tags:` | Ansible `tags:` on tasks/plays |
|--|------------------|--------------------------------|
| Purpose | Catalog classification (pillar / lifecycle / maturity) | Runtime selection (`--tags`, AAP job slicing) |
| Scope | Entire playbook file | Individual tasks or plays |
| This spec | Required on all playbooks | Unchanged |

**Do not mirror header tags into Ansible runtime tags.** Header tags exist for
catalog alignment, grep, and future tooling. They are not copied to play- or
task-level `tags:` unless a separate change explicitly requires runtime tag
filtering (out of scope for header rollout). Existing runtime tags (e.g.
`always`, role-specific tags) stay as-is.

**Rollout (playbooks)**

Existing playbooks may use legacy banner styles or omit headers. New and
updated playbooks must conform. Bulk playbook header additions are tracked in
batched PRs; this section is the source of truth for format and vocabulary.

## 5. Role Design

- **Self-contained.** Each role includes `tasks/`, `defaults/main.yml`,
  `handlers/` (if needed), `templates/`, `files/`, and `README.md`.
- **Parameterized.** Where multiple legacy playbooks did the same thing with
  minor variations, they are consolidated into one role with a mode/action
  parameter (e.g. `win_service_action: start | stop | restart | check`).
- **GPA variable naming.** Follow Red Hat COP Good Practices for Ansible:
  - **Public API** (defaults, extra vars, survey parameters): `{rolename}_*`
    (e.g. `firewall_management_zones`, not `zones`).
  - **Internal / private** (facts, computed values, registered results kept
    inside the role): `__{rolename}_*` — **double underscore**
    (e.g. `__web_service_rca_profile`, not `_web_rca_profile`).
- **Scenario `group_vars` absorbed.** Any `group_vars/` from legacy scenarios
  are merged into `roles/<role>/defaults/main.yml` during migration.

### Role README metadata

Roles ship in this repository alongside playbooks and are documented in
`roles/<role>/README.md` (GPA). Use the **same catalog classification** as
playbook headers (§4 — pillar / lifecycle / maturity), as **Markdown**
immediately under the `# Role:` title — not as a YAML comment block in
`tasks/main.yml`.

**Required lines** (when adding or updating a role README):

| Line | Required | Notes |
|------|:--------:|-------|
| `**Tags:**` | Yes | Prefixed tags; must include `pillar:`, `lifecycle:`, `maturity:` |
| `**Supported OS:**` | When restricted | Same rule as playbooks — include when assert or role scope restricts OS |
| Body | Yes | Existing GPA sections (Requirements, Role Variables, Example Playbook, etc.) |

**Example** (top of `roles/bind_primary_dns/README.md`):

```markdown
# Role: `bind_primary_dns`

**Tags:** pillar:network,lifecycle:day1,maturity:l2-process-oriented,platform:linux,tech:bind
**Supported OS:** RHEL/CentOS 7/8/9

Deploy and configure a BIND primary DNS server...
```

Playbook and role tags may differ slightly: the playbook header describes the
**entry point** (inventory, multi-play); the role README describes **role
behaviour**. One playbook may invoke several roles — each role documents itself
at its own matrix classification.

**Rollout (roles)**

Bulk README metadata additions follow playbook header batches (same PR or the
next). New and updated roles must include Tags (and Supported OS when
applicable) in README.

## 6. Consolidation Decisions

| Family | Source scenarios | Result | Key parameter |
|--------|-----------------|--------|---------------|
| Web service RCA | 17 (httpd) + 18 (nginx) | 1 role `web_service_rca` | `web_service_rca_service: httpd \| nginx` |
| Memory RCA | 21 (high mem) + 22 (OOM) | 1 role `memory_rca` | `memory_rca_mode: high_utilization \| oom` |
| Diagnostic RCA | 29, 30, 31, 32 | 1 role `linux_diagnostic_rca` | `diagnostic_check_type: zombie_process \| disk_io \| systemd_unit \| privileged_account` |
| Yum repo client | 04 (client path) | 1 role `yum_repo_client` | `yum_repo_client_repositories` (list of `yum_repository` dicts) |
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

Playbooks that produce diagnostic or audit reports follow
`docs/reporting-standard.md`:

- Each reporting role writes plaintext under `/var/lib/ansible-reports/` on
  the managed host (overwrite each run) and prints the same body with
  `debug` so the AAP job log is the primary delivery mechanism.
- Variables are role-prefixed. There is no shared archive-publish role and
  no repo-wide `report_publish_enabled` toggle.

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
