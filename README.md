## ACL Ready-to-Implement Content

Ansible Content Library (ACL) — production-ready automation content maintained by the
Ansible Services Platform Team.

This repository follows the
[Ansible Content Library specification](https://docs.google.com/document/d/1eJmRoc5Hwz0jhGYHpQaU4VDHmCOy-d3kNyXjIXBnlq0)
and [Red Hat COP Good Practices for Ansible (GPA)](https://redhat-cop.github.io/automation-good-practices/).

---

## Repository Structure

```
acl-ready-to-implement-content/
├── playbooks/              # Flat thin playbooks (linux-*.yml / win-*.yml)
├── roles/                  # Shared roles (logic, defaults, templates, READMEs)
├── collections/
│   └── requirements.yml
├── execution_environments/
│   ├── execution-environment.yml
│   ├── requirements.yml
│   ├── requirements.txt
│   └── bindep.txt
├── controller_config/      # AAP as-code (infra.controller_configuration)
│   ├── projects.yml
│   ├── job_templates.yml
│   └── README.md
├── inventory/
│   ├── inventory.example
│   └── group_vars/all/reporting.yml
├── docs/
│   ├── getting-started.md
│   ├── architecture.md
│   ├── aap-deployment-guide.md
│   ├── reporting-standard.md
│   └── specifications.md
└── .cursor/skills/         # AI assistant guidelines
```

## Principles

1. **AAP 2.6+** — all content tested on current supported AAP versions.
2. **Inventory and CaC live in AAP** — playbooks stay thin; operator config is in inventory or surveys.
3. **Minimal configuration** — sensible defaults; survey overrides for site-specific values.
4. **Supported RHEL targets only** — RHEL in a supportable state (including ELS/EUS).
5. **Report publishing standard** — see [docs/reporting-standard.md](docs/reporting-standard.md).

## Getting Started

See [docs/getting-started.md](docs/getting-started.md) for clone, collection install,
inventory setup, and run instructions.

Quick reference:

```bash
# Install shared collections
ansible-galaxy collection install -r collections/requirements.yml

# Run a playbook (from repo root)
ansible-playbook playbooks/linux-dns-primary.yml -i inventory/lab.ini

# Lint
pip install -r requirements-dev.txt
pre-commit install
pre-commit run --all-files
```

Architecture and migration context: [docs/architecture.md](docs/architecture.md).  
AAP deployment: [docs/aap-deployment-guide.md](docs/aap-deployment-guide.md).

---

## Playbooks

Thin entry points under [`playbooks/`](playbooks/). Task logic lives in [`roles/`](roles/).
Run each playbook from the repository root so `ansible.cfg` resolves `roles_path`.

### Linux — DNS, DHCP, and name services

| Playbook | Description |
|----------|-------------|
| [linux-dns-primary.yml](playbooks/linux-dns-primary.yml) | Deploy and configure a BIND primary (master) DNS server |
| [linux-dns-secondary.yml](playbooks/linux-dns-secondary.yml) | Deploy and configure a BIND secondary (slave) DNS server |
| [linux-dns-unbound.yml](playbooks/linux-dns-unbound.yml) | Deploy and configure an Unbound caching DNS resolver |
| [linux-dns-sync.yml](playbooks/linux-dns-sync.yml) | Synchronize DNS resolver configuration across hosts |
| [linux-dhcp-server.yml](playbooks/linux-dhcp-server.yml) | Deploy and configure DHCPv4/DHCPv6 server |
| [linux-dhcp-client.yml](playbooks/linux-dhcp-client.yml) | Configure DHCP client via `rhel-system-roles.network` |

### Linux — Configuration and deployment

| Playbook | Description |
|----------|-------------|
| [linux-selinux-config.yml](playbooks/linux-selinux-config.yml) | Configure SELinux state and related settings |
| [linux-yum-repo-client.yml](playbooks/linux-yum-repo-client.yml) | Configure yum/dnf repository entries on RHEL clients |
| [linux-firewall-management.yml](playbooks/linux-firewall-management.yml) | Manage firewalld zones, services, and ports |
| [linux-nfs-server.yml](playbooks/linux-nfs-server.yml) | Deploy and configure an NFS server and exports |
| [linux-mariadb-config.yml](playbooks/linux-mariadb-config.yml) | Deploy and configure MariaDB with hardening and backup |
| [linux-postgresql-deploy.yml](playbooks/linux-postgresql-deploy.yml) | Simple PostgreSQL server deployment |
| [linux-rhel9-standard-config.yml](playbooks/linux-rhel9-standard-config.yml) | RHEL 9 standardization bundle (users, firewall, chrony, and more) |
| [linux-vnc-config.yml](playbooks/linux-vnc-config.yml) | Configure TigerVNC server |
| [linux-kdump-config.yml](playbooks/linux-kdump-config.yml) | Configure kdump kernel crash dump service |
| [linux-cve-patching.yml](playbooks/linux-cve-patching.yml) | Install security patches based on CVE ID |
| [linux-networkmanager.yml](playbooks/linux-networkmanager.yml) | Manage NetworkManager connections and devices |
| [linux-openstack-vm-lifecycle.yml](playbooks/linux-openstack-vm-lifecycle.yml) | OpenStack VM lifecycle operations (create, power, resize, delete) |

### Linux — Inspection, reporting, and troubleshooting

| Playbook | Description |
|----------|-------------|
| [linux-os-info-report.yml](playbooks/linux-os-info-report.yml) | Collect OS information and generate a report |
| [linux-service-inspection.yml](playbooks/linux-service-inspection.yml) | Inspect critical Linux service status and generate reports |
| [linux-filesystem-check.yml](playbooks/linux-filesystem-check.yml) | Check filesystem health and generate reports |
| [linux-kernel-log-check.yml](playbooks/linux-kernel-log-check.yml) | Monitor kernel logs for errors and generate reports |
| [linux-sshd-troubleshoot.yml](playbooks/linux-sshd-troubleshoot.yml) | SSH connection troubleshooting and auto-remediation |
| [linux-network-fault-diagnosis.yml](playbooks/linux-network-fault-diagnosis.yml) | Network latency diagnosis with interface, system, and traffic analysis |
| [linux-pcp-analysis.yml](playbooks/linux-pcp-analysis.yml) | Install and run Performance Co-Pilot (PCP) analysis |
| [linux-ssl-cert-check.yml](playbooks/linux-ssl-cert-check.yml) | Check SSL certificate expiry and generate a report |
| [linux-xfs-health-check.yml](playbooks/linux-xfs-health-check.yml) | XFS filesystem health inspection |

### Linux — Root cause analysis (RCA)

| Playbook | Description |
|----------|-------------|
| [linux-web-service-rca.yml](playbooks/linux-web-service-rca.yml) | Web service fault diagnosis (httpd/nginx) |
| [linux-chronyd-rca.yml](playbooks/linux-chronyd-rca.yml) | Chronyd time-sync fault diagnosis |
| [linux-ftp-rca.yml](playbooks/linux-ftp-rca.yml) | FTP service fault diagnosis |
| [linux-memory-rca.yml](playbooks/linux-memory-rca.yml) | Memory diagnosis (high utilization / OOM) |
| [linux-zombie-process-rca.yml](playbooks/linux-zombie-process-rca.yml) | Zombie process root cause analysis |
| [linux-disk-io-rca.yml](playbooks/linux-disk-io-rca.yml) | Disk I/O pressure root cause analysis |
| [linux-systemd-check.yml](playbooks/linux-systemd-check.yml) | Systemd unit deep check and analysis |
| [linux-privileged-account-audit.yml](playbooks/linux-privileged-account-audit.yml) | Privileged account and sudo security audit |

### Windows

| Playbook | Description |
|----------|-------------|
| [win-service-management.yml](playbooks/win-service-management.yml) | Manage Windows services (check, start, stop, restart) |
| [win-iis-deploy.yml](playbooks/win-iis-deploy.yml) | Deploy and configure IIS |
| [win-software-management.yml](playbooks/win-software-management.yml) | Install or remove Windows software packages |
| [win-chocolatey.yml](playbooks/win-chocolatey.yml) | Manage packages via Chocolatey |
| [win-patch-management.yml](playbooks/win-patch-management.yml) | Manage Windows patches and updates |

Role-level variables and usage notes are documented in each role’s `README.md` under [`roles/`](roles/).

---

## Support

This content is supported by the Ansible Services Platform Team at Red Hat.
