# Ansible Report Publishing Standard

This document defines the repo-wide pattern for generating and publishing
reports from Ansible Automation Platform (AAP) jobs.

## Problem

On AAP, `delegate_to: localhost` and `ansible.builtin.fetch` write to the
execution environment container. That filesystem is ephemeral and reports are
lost when the job ends.

## Solution

1. Write reports on **managed hosts** under `/var/lib/ansible-reports/<namespace>/`
2. Emit a **plaintext summary** to job output via `debug` (last tasks in Play 1)
3. **Publish** reports to a persistent archive server using the shared
   `report_publish` role

## Variable hierarchy (GPA-aligned)

Operator toggles live in **inventory**, not playbooks. Playbooks stay thin;
survey answers override everything at runtime.

| Priority (high → low) | Source | Example |
|----------------------|--------|---------|
| 1 | Survey / job extra vars | `generate_html: true` from AAP survey |
| 2 | Play `vars:` | `report_namespace: os_info` (playbook-specific) |
| 3 | `vars_files` / inventory `group_vars` | `inventory/group_vars/all/reporting.yml` |
| 4 | Role `defaults/` | `report_namespace` in content-role defaults |

**Canonical defaults file** (opt-in publishing, no HTML unless requested):

[`inventory/group_vars/all/reporting.yml`](../inventory/group_vars/all/reporting.yml)

```yaml
generate_html: false
report_publish_enabled: false
```

Each reporting playbook loads this file on Play 1 via `vars_files`. The same
file applies when the inventory source is attached on AAP. Do not duplicate
these toggles in play `vars:` or per-project `group_vars/`.

## AAP job template configuration

### Extra vars / survey (override inventory defaults)

| Variable | Survey? | Default | Purpose |
|----------|---------|---------|---------|
| `report_archive_host` | no | — | Required when `report_publish_enabled` is true |
| `report_publish_enabled` | yes | `false` | Enable archive publishing |
| `generate_html` | yes | `false` | Generate HTML files on targets |
| `report_publish_web_root` | no | `/var/www/ansible-reports` | Archive filesystem path |
| `report_publish_url_base` | no | `http://<host>/ansible-reports` | URL in job output |

The archive server is **not** part of the job inventory. Operators pass only
the hosts to inspect.

## Standard variables (Set 1 — report generation)

| Variable | Default | Purpose |
|----------|---------|---------|
| `report_namespace` | (required) | Unique folder name per playbook |
| `report_local_dir` | `/var/lib/ansible-reports/{{ report_namespace }}` | Staging on managed host |
| `report_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique job folder |
| `report_text_path` | `.../{{ inventory_hostname }}-{{ report_job_id }}.txt` | Required plaintext report |
| `report_html_path` | `.../{{ inventory_hostname }}-{{ report_job_id }}.html` | Optional HTML report |
| `report_json_path` | `.../{{ inventory_hostname }}-{{ report_job_id }}.json` | Optional JSON report |
| `generate_html` | `false` (inventory) | Gate all HTML template tasks |
| `report_publish_enabled` | `false` (inventory) | Gate Play 2 publishing |

## Playbook structure (two plays)

```yaml
# Play 1: hosts: all
  vars_files:
    - ../inventory/group_vars/all/reporting.yml
  vars:
    report_namespace: <playbook-specific>
  tasks:
    # ... collect, write report_text_path, debug summary ...
    - name: Remember report publishing settings for Play 2
      ansible.builtin.set_fact:
        report_source_hosts: "{{ ansible_play_hosts_all }}"
        report_publish_enabled: "{{ report_publish_enabled }}"
      run_once: true
      delegate_to: localhost
      delegate_facts: true

# Play 2: import shared publish play (skipped when publishing is disabled)
- name: Publish reports to archive host
  ansible.builtin.import_playbook: ../roles/report_publish/playbooks/publish.yml
  vars:
    report_namespace: <same as Play 1>
  when: hostvars['localhost'].report_publish_enabled | bool
```

### Why two plays?

- **Play 1** runs on your managed hosts (the job inventory).
- **Play 2** connects to `report_archive_host`, which is not in the job inventory.

## HTML generation

All HTML `ansible.builtin.template` tasks must use:

```yaml
when: generate_html | bool
```

Publishing collects `hostname-jobid.*` — plaintext `.txt` is always published
when publishing is enabled; HTML is included only when generated.

## Archive host preparation (one-time)

On the archive server (`report_archive_host`):

```bash
sudo mkdir -p /var/www/ansible-reports
sudo chown nginx:nginx /var/www/ansible-reports
sudo chmod 755 /var/www/ansible-reports
```

Example nginx configuration:

```nginx
location /ansible-reports/ {
    alias /var/www/ansible-reports/;
    autoindex off;
}
```

If SELinux is enforcing:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t '/var/www/ansible-reports(/.*)?'
sudo restorecon -Rv /var/www/ansible-reports
```

## Local testing

```bash
ansible-playbook playbook.yml -i inventory
# publishing off by default; enable explicitly:
ansible-playbook playbook.yml -i inventory \
  -e report_archive_host=reports.example.net \
  -e report_publish_enabled=true
```

## Deprecated variables

| Old variable | Replacement |
|--------------|-------------|
| `report_output_dir` | `report_local_dir` |
| `report_output_path` | `report_text_path` |
| `report_path` | `report_text_path` or `report_json_path` |
| `report_dest_path` | removed (use `report_publish` role) |
| `*_collection_dest_path` | removed (use `report_publish` role) |
| `report_delivery_method` | removed (use AAP notifications + archive URL) |
| `[report_archive]` inventory group | `report_archive_host` extra var |
| Toggles in play `vars:` / role defaults | `inventory/group_vars/all/reporting.yml` |

## Role reference

Shared role: [`roles/report_publish/`](../roles/report_publish/)

Shared publish play: [`roles/report_publish/playbooks/publish.yml`](../roles/report_publish/playbooks/publish.yml)

Tasks: validate, collect (fetch + copy), index, summary, cleanup.
