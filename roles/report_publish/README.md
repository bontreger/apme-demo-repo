# Role: `report_publish`

Collects diagnostic reports from target hosts, publishes them to a central archive web server with an HTML index page, and cleans up reports older than a configurable retention period.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `report_publish_source_hosts` | `{{ hostvars['localhost'].report_source_hosts \| default([]) }}` | List of hosts that generated reports in the diagnostic play |
| `report_publish_namespace` | `{{ report_namespace \| default('') }}` | Report category namespace for directory organization |
| `report_publish_job_id` | `{{ report_job_id \| default(awx_job_id \| default(ansible_date_time.epoch)) }}` | Unique job identifier used in directory paths |
| `report_publish_archive_host` | `{{ report_archive_host }}` | Hostname of the central archive server (required extra var) |
| `report_publish_web_root` | `/var/www/ansible-reports` | Web root directory on the archive host |
| `report_publish_url_base` | `http://{{ report_archive_host }}/ansible-reports` | Base URL for accessing published reports |
| `report_publish_dest_dir` | `{{ report_publish_web_root }}/{{ report_publish_namespace }}/{{ report_publish_job_id }}` | Destination directory on the archive host |
| `report_publish_url` | `{{ report_publish_url_base }}/{{ report_publish_namespace }}/{{ report_publish_job_id }}/` | Full URL to the published report directory |
| `report_publish_staging_dir` | `/tmp/ansible-report-staging/{{ report_publish_job_id }}` | Temporary staging directory for report collection |
| `report_publish_retention_days` | `30` | Number of days to retain old reports before cleanup |
| `report_publish_run_cleanup` | `true` | Whether to run cleanup of old reports on the archive host |
| `report_publish_skip_collect` | `false` | Skip collecting reports from targets (for aggregated reports written directly on archive) |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Inspect critical Linux service status and generate reports
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: service_check

- name: Publish reports to archive host
  ansible.builtin.import_playbook: ../roles/report_publish/playbooks/publish.yml
  vars:
    report_namespace: service_check
  when: (hostvars['localhost'].report_publish_enabled | default(false)) | bool
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
