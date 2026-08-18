# ACL Ready-to-Implement Content Restructuring Plan

Migrate all remaining scenarios from `ready-to-use-content` into `acl-ready-to-implement-content` following ACL spec and GPA principles: flat `playbooks/` layout, shared top-level `roles/`, thin playbooks calling parameterized roles, platform-prefixed naming (`linux-`/`win-`). Exclude 200 and 301 as ADUC candidates.

## Current State

**Already migrated** (6 scenarios, 17 roles):
- `10_dns_config` -> `playbooks/dns-config/`
- `11_dhcp_config` -> `playbooks/dhcp-config/`
- `12_maria_db_config` -> `playbooks/mariadb-config/`
- `15_rhel9_standard_config` -> `playbooks/rhel9-standard-config/`
- `16_network_fault_diagnosis` -> `playbooks/network-fault-diagnosis/`
- `302_openstack_vm_lifecycle` -> `playbooks/openstack-vm-lifecycle/`
- `101/102/103` -> `docs/vmware/` (docs only)

**Excluded (ADUC repos)**: `200_virt-automation-vmware_and_ocpv`, `301_convert2rhel-automation`

**Remaining to migrate**: 33 scenarios (01-09, 13-14, 17-33, 23-27, 81-85)

---

## Target Directory Structure

Playbooks are **flat .yml files** directly under `playbooks/` -- no subdirectories. All logic, defaults, templates, and documentation live in the role.

```
acl-ready-to-implement-content/
├── playbooks/                         # flat .yml files, no subdirectories
│   ├── linux-dns-primary.yml          # existing, flattened + renamed
│   ├── linux-dns-secondary.yml        # existing, flattened + renamed
│   ├── linux-dns-unbound.yml          # existing, flattened + renamed
│   ├── linux-dhcp-server.yml          # existing, flattened + renamed
│   ├── linux-dhcp-client.yml          # existing, flattened + renamed
│   ├── linux-mariadb-config.yml       # existing, flattened + renamed
│   ├── linux-rhel9-standard-config.yml  # existing, flattened + renamed
│   ├── linux-network-fault-diagnosis.yml # existing, flattened + renamed
│   ├── linux-openstack-vm-lifecycle.yml  # existing, flattened + renamed
│   ├── linux-os-info-report.yml       # from 01
│   ├── linux-dns-sync.yml             # from 02
│   ├── linux-selinux-config.yml       # from 03
│   ├── linux-yum-repo-client.yml      # from 04 (client-only)
│   ├── linux-firewall-management.yml  # from 05
│   ├── linux-service-inspection.yml   # from 06
│   ├── linux-filesystem-check.yml     # from 07
│   ├── linux-kernel-log-check.yml     # from 08
│   ├── linux-networkmanager.yml       # from 09
│   ├── linux-nfs-server.yml           # from 13
│   ├── linux-sshd-troubleshoot.yml    # from 14
│   ├── linux-web-service-rca.yml      # consolidated from 17+18
│   ├── linux-chronyd-rca.yml          # from 19
│   ├── linux-ftp-rca.yml              # from 20
│   ├── linux-memory-rca.yml           # consolidated from 21+22
│   ├── linux-postgresql-deploy.yml    # from 23
│   ├── linux-pcp-analysis.yml         # from 24
│   ├── linux-vnc-config.yml           # from 25
│   ├── linux-kdump-config.yml         # from 26
│   ├── linux-cve-patching.yml         # from 27
│   ├── linux-xfs-health-check.yml     # from 28
│   ├── linux-zombie-process-rca.yml   # from 29
│   ├── linux-disk-io-rca.yml          # from 30
│   ├── linux-systemd-check.yml        # from 31
│   ├── linux-privileged-account-audit.yml # from 32
│   ├── linux-ssl-cert-check.yml       # from 33
│   ├── win-service-management.yml     # consolidated from 81 (6 -> 1)
│   ├── win-iis-deploy.yml             # from 82 install
│   ├── win-software-management.yml    # consolidated from 83 (2 -> 1)
│   ├── win-chocolatey.yml             # consolidated from 84 (2 -> 1)
│   └── win-patch-management.yml       # consolidated from 85 (3 -> 1)
├── roles/                             # all roles shared at top level
│   ├── ... (existing 17 roles, each with README.md)
│   ├── os_info_report/
│   ├── dns_sync/
│   ├── selinux_config/
│   ├── yum_repo_client/
│   ├── firewall_management/
│   ├── service_inspection/
│   ├── filesystem_check/
│   ├── kernel_log_monitor/
│   ├── networkmanager_mgmt/
│   ├── nfs_server/
│   ├── sshd_troubleshoot/
│   ├── web_service_rca/               # consolidated 17+18
│   ├── chronyd_rca/
│   ├── ftp_rca/
│   ├── memory_rca/                    # consolidated 21+22
│   ├── postgresql_server/
│   ├── pcp_analysis/
│   ├── vnc_config/
│   ├── kdump_config/
│   ├── cve_patching/
│   ├── xfs_health_check/
│   ├── linux_diagnostic_rca/          # shared scaffolding for 29-32
│   ├── ssl_cert_check/
│   ├── win_service_management/        # consolidated 81
│   ├── win_iis_deploy/
│   ├── win_software_management/       # consolidated 83
│   ├── win_chocolatey/                # consolidated 84
│   └── win_patch_management/          # consolidated 85
├── collections/requirements.yml       # updated with windows collections
├── controller_config/                 # updated with all job templates
├── execution_environments/            # updated for windows support
├── inventory/
├── docs/
└── ...
```

**Key principles:**
- Each playbook is a single flat `.yml` file -- a thin entry point calling one or more roles
- All task logic, defaults, templates, and `README.md` documentation live inside the role directory
- Scenario-specific variables go into `roles/<role>/defaults/main.yml`
- No `group_vars/` or `templates/` outside of roles (except `inventory/group_vars/` for repo-wide settings)

---

## Phase 1: Flatten and Rename Existing Playbooks

Convert the 6 existing playbook subdirectories into flat `.yml` files with `linux-` prefix. Move any `group_vars/` content into role `defaults/main.yml`. Delete the now-empty subdirectories.

| Current Directory + File | New Flat File |
|--------------------------|---------------|
| `playbooks/dns-config/deploy_bind_primary_site.yml` | `playbooks/linux-dns-primary.yml` |
| `playbooks/dns-config/deploy_bind_secondary_site.yml` | `playbooks/linux-dns-secondary.yml` |
| `playbooks/dns-config/deploy_unbound_site.yml` | `playbooks/linux-dns-unbound.yml` |
| `playbooks/dhcp-config/deploy_dhcp_server_site.yml` | `playbooks/linux-dhcp-server.yml` |
| `playbooks/dhcp-config/configure_dhcp_client_site.yml` | `playbooks/linux-dhcp-client.yml` |
| `playbooks/mariadb-config/deploy_mariadb_site.yml` | `playbooks/linux-mariadb-config.yml` |
| `playbooks/rhel9-standard-config/rhel9_standardization_site.yml` | `playbooks/linux-rhel9-standard-config.yml` |
| `playbooks/network-fault-diagnosis/network_diagnosis_site.yml` | `playbooks/linux-network-fault-diagnosis.yml` |
| `playbooks/openstack-vm-lifecycle/site.yml` | `playbooks/linux-openstack-vm-lifecycle.yml` |

Also handle:
- `playbooks/network-fault-diagnosis/group_vars/all.yml` -> merge into `roles/report_generation/defaults/main.yml` or `inventory/group_vars/all/reporting.yml`
- `playbooks/rhel9-standard-config/group_vars/all.yml` -> merge into appropriate role defaults
- Update `controller_config/job_templates.yml` playbook paths to match new flat filenames
- Fix `import_playbook` reference in network diagnosis (currently `../../roles/report_publish/playbooks/publish.yml` -- adjust relative path)
- Remove empty subdirectory READMEs; ensure each role has its own `README.md`

---

## Phase 2: Migrate Standalone Linux Scenarios (No Consolidation)

These source scenarios have no roles -- all logic is in one monolithic playbook. For each, extract the task logic into a new role under `roles/`, then create a thin playbook.

**Thin playbook pattern:**

```yaml
# playbooks/linux-<name>.yml
---
- name: <Descriptive play name>
  hosts: "{{ survey_target_hosts | default('all_rhel') }}"
  become: true
  gather_facts: true
  pre_tasks:
    - name: Validate RHEL compatibility
      ansible.builtin.assert: ...
  roles:
    - role: <role_name>
```

| Source | Playbook File | Role Name | Notes |
|--------|--------------|-----------|-------|
| `01_os_info_collection_report` | `linux-os-info-report.yml` | `os_info_report` | Collects OS facts, generates report |
| `02_dns_config_and_sync` | `linux-dns-sync.yml` | `dns_sync` | DNS resolver configuration |
| `03_unified_selinux_configuration` | `linux-selinux-config.yml` | `selinux_config` | SELinux mode/booleans/contexts |
| `05_firewall_mgmt` | `linux-firewall-management.yml` | `firewall_management` | firewalld zone/service/port management |
| `13_nfs_server_config` | `linux-nfs-server.yml` | `nfs_server` | NFS server setup + exports |
| `14_sshd_fix` | `linux-sshd-troubleshoot.yml` | `sshd_troubleshoot` | SSH connectivity diagnosis and repair |
| `23_postgresql_server_deploy` | `linux-postgresql-deploy.yml` | `postgresql_server` | PostgreSQL simple deployment |
| `24_os_pcp` | `linux-pcp-analysis.yml` | `pcp_analysis` | PCP performance analysis |

**Special case -- `04_yum_config` (client path extracted):**

Source had `yum_repo_server_tasks.yml`, `yum_repo_client_tasks.yml`,
`yum_repo_deployment.yml`. The client path is now a dedicated function role;
server hosting is deferred.

- **Role**: `yum_repo_client` with `yum_repo_client_repositories` (list of
  `yum_repository` dicts)
- **Playbook**: `playbooks/linux-yum-repo-client.yml`
- Server ISO/httpd hosting is out of scope for this role (follow-up
  `yum_repo_server` if retained).

---

## Phase 3: Migrate Linux Scenarios with Existing Roles

These source scenarios already have `roles/` subdirectories. Hoist their roles to the top-level `roles/` directory, create thin playbooks.

| Source | Playbook File | Roles to Hoist | Notes |
|--------|--------------|----------------|-------|
| `06_automated_inspection_of_linux` | `linux-service-inspection.yml` | `service_inspection` (+ any sub-roles) | Linux service health checks |
| `07_linux_fs_auto_check` | `linux-filesystem-check.yml` | `filesystem_check` | Filesystem health validation |
| `08_kernel_log_error_check` | `linux-kernel-log-check.yml` | `kernel_log_monitor` | Kernel log error scanning |
| `09_networkmanager_auto_manage` | `linux-networkmanager.yml` | `networkmanager_mgmt` | NetworkManager interface management |

Check for role name conflicts with already-migrated roles before hoisting.

---

## Phase 4: Consolidate RCA/Diagnosis Families into Parameterized Roles

### 4a. Web Service RCA (17 + 18 -> 1 role)

Source `httpd_diagnosis_site.yml` (475 lines) and `nginx_diagnosis_site.yml` (269 lines) share ~70% identical structure.

- **Role**: `web_service_rca`
- **Playbook**: `playbooks/linux-web-service-rca.yml`
- **Key parameter**: `web_service_rca_service: httpd | nginx`
- **Role design**:
  - `defaults/main.yml`: service profiles dict mapping service name to package, config path, syntax check command, log paths, ports, SELinux booleans
  - `tasks/main.yml`: shared flow (install check -> service status -> syntax check -> config analysis -> log analysis -> report)
  - `tasks/httpd_checks.yml`, `tasks/nginx_checks.yml`: service-specific diagnostic commands
  - `templates/web_service_rca_report.j2`: unified report template with conditional sections

### 4b. Chronyd RCA (19 -> standalone)

Different enough from web service RCA (time-sync-specific commands like `chronyc`, `timedatectl`).

- **Role**: `chronyd_rca`
- **Playbook**: `playbooks/linux-chronyd-rca.yml`
- Move `templates/` content into role

### 4c. FTP RCA (20 -> standalone)

Server/client split makes this architecturally distinct.

- **Role**: `ftp_rca`
- **Playbook**: `playbooks/linux-ftp-rca.yml`
- Move `templates/` content into role

### 4d. Memory RCA (21 + 22 -> 1 role)

Source `troubleshooting04_HignMemUsage.yml` (192 lines) and `troubleshooting03_oom.yml` (217 lines) share memory diagnostic commands.

- **Role**: `memory_rca`
- **Playbook**: `playbooks/linux-memory-rca.yml`
- **Key parameter**: `memory_rca_mode: high_utilization | oom`
- **Role design**:
  - `tasks/main.yml`: shared checks (free, /proc/meminfo, dmidecode, vmstat)
  - `tasks/high_utilization.yml`: top, ps aux, sar, iostat, lsof
  - `tasks/oom.yml`: OOM grep in logs, SAR history, journalctl
  - `templates/memory_rca_report.j2`: unified template

### 4e. Diagnostic Reports (29-32 -> 1 shared role, 28 + 33 -> standalone)

Scenarios 29-32 share ~70-80% boilerplate (OS assert, block/rescue, JSON+plaintext report, report_publish). Each has unique diagnostic commands.

- **Role**: `linux_diagnostic_rca`
- **Key parameter**: `diagnostic_check_type: zombie_process | disk_io | systemd_unit | privileged_account`
- **Role design**:
  - `tasks/main.yml`: shared scaffolding (OS assert, report dir, JSON/txt output, publish)
  - `tasks/checks/zombie_process.yml`, `tasks/checks/disk_io.yml`, etc.
  - Each check type included via `include_tasks: "checks/{{ diagnostic_check_type }}.yml"`
- **Playbooks**: One per check type for discoverability:
  - `playbooks/linux-zombie-process-rca.yml` (sets `diagnostic_check_type: zombie_process`)
  - `playbooks/linux-disk-io-rca.yml` (sets `diagnostic_check_type: disk_io`)
  - `playbooks/linux-systemd-check.yml` (sets `diagnostic_check_type: systemd_unit`)
  - `playbooks/linux-privileged-account-audit.yml` (sets `diagnostic_check_type: privileged_account`)

**Standalone (different architecture):**
- **28 XFS** -> `xfs_health_check` role, `playbooks/linux-xfs-health-check.yml` (different report assembly pattern using `lineinfile`)
- **33 SSL** -> `ssl_cert_check` role, `playbooks/linux-ssl-cert-check.yml` (runs on localhost, uses `community.crypto`, Markdown output)

---

## Phase 5: Migrate Config Scenarios

No consolidation possible -- these are unrelated domains.

| Source | Playbook File | Role Name |
|--------|--------------|-----------|
| `25_vnc_config` | `linux-vnc-config.yml` | `vnc_config` |
| `26_kdump_config` | `linux-kdump-config.yml` | `kdump_config` |
| `27_cve_based_patching` | `linux-cve-patching.yml` | `cve_patching` |

Each role gets its own `defaults/main.yml` with survey-friendly variable names.

---

## Phase 6: Consolidate and Migrate Windows Scenarios

### 6a. Service Management (81: 6 playbooks -> 1 role)

All 6 playbooks call `ansible.windows.win_service` with the same `win_services` list, differing only by `state` and `start_mode`.

- **Role**: `win_service_management`
- **Playbook**: `playbooks/win-service-management.yml`
- **Key parameters**: `win_service_action: check | start | stop | restart`, `win_service_start_mode: auto | manual | absent`
- **Role design**:
  - `defaults/main.yml`: `win_service_action: check`, `win_services: []`
  - `tasks/main.yml`: single `ansible.windows.win_service` loop with `state` and `start_mode` set by variables

### 6b. IIS (82: 3 playbooks -> 1 role + reuse)

- **Role**: `win_iis_deploy` (installation: feature install, firewall, index page, health check)
- **Playbook**: `playbooks/win-iis-deploy.yml`
- IIS start/stop reuses `win_service_management` role with `win_services: [{name: W3Svc}]`

### 6c. Software Management (83: 2 playbooks -> 1 role)

- **Role**: `win_software_management`
- **Playbook**: `playbooks/win-software-management.yml`
- **Key parameter**: `win_software_state: present | absent`
- Structured `win_software_packages[]` list with per-package `path`, `product_id`, `arguments`

### 6d. Chocolatey (84: 2 playbooks -> 1 role)

- **Role**: `win_chocolatey`
- **Playbook**: `playbooks/win-chocolatey.yml`
- **Key parameter**: `win_chocolatey_state: present | absent`
- `win_chocolatey_packages: [notepadplusplus, git]`

### 6e. Patch Management (85: 3 playbooks -> 1 role)

- **Role**: `win_patch_management`
- **Playbook**: `playbooks/win-patch-management.yml`
- **Key parameter**: `win_patch_mode: install_all | install_kb | report`
- `win_patch_accept_list: []` (for KB-specific installs)
- Consolidated template for Windows Update report

---

## Phase 7: Infrastructure Updates

### collections/requirements.yml
Add missing collections needed by new scenarios:
- `ansible.windows` (Windows scenarios)
- `chocolatey.chocolatey` (84)
- `community.crypto` (33 SSL checks)
- Any others discovered during migration

### execution_environments/
- Update `requirements.yml` to include Windows collections
- Update `bindep.txt` / `requirements.txt` if new Python deps are needed

### controller_config/job_templates.yml
Add job template entries for all new scenarios. Each template references its `playbooks/<name>.yml` path and defines relevant survey variables.

### controller_config/configure_aap.yml
Verify all new templates are registered.

### README.md
Update the scenario listing table with all migrated playbooks, organized by platform (Linux / Windows).

### ansible-lint
Run `ansible-lint` with production profile against all new content. Fix any violations before committing.

---

## Consolidation Impact Summary

| Family | Source Playbooks | Target Playbooks | Target Roles | Reduction |
|--------|-----------------|-----------------|--------------|-----------|
| Service mgmt (81) | 6 | 1 | 1 | 6 -> 1 |
| Chocolatey (84) | 2 | 1 | 1 | 2 -> 1 |
| Software mgmt (83) | 2 | 1 | 1 | 2 -> 1 |
| Patch mgmt (85) | 3 | 1 | 1 | 3 -> 1 |
| Web service RCA (17+18) | 2 | 1 | 1 | 2 -> 1 |
| Memory RCA (21+22) | 2 | 1 | 1 | 2 -> 1 |
| Diagnostic RCA (29-32) | 4 | 4 | 1 shared | 4 roles -> 1 |
| Yum repo (04) | 3 | 1 | 1 | 3 -> 1 |
| **Totals** | **24** | **11** | **8** | **~55% reduction** |

---

## Out of Scope

- `200_virt-automation-vmware_and_ocpv` -- separate ADUC repo
- `301_convert2rhel-automation` -- separate ADUC repo
- `101/102/103` VMware docs -- already migrated under `docs/vmware/`
