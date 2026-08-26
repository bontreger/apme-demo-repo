# Role: `hosts_config`

**Tags:** pillar:os,lifecycle:day1,maturity:l2,platform:linux,context:rhel9-only  
**Supported OS:** RHEL 9 only

Standardizes /etc/hosts across target hosts by fetching a template from a designated template server, merging it, and ensuring the local host entry is present.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- A `template_servers` inventory group or `template_server` variable defining the source host

## Role Variables

This role does not define variables in `defaults/main.yml`. It relies on the following inventory/group variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `template_server.hosts_file` | *(required)* | Path to the hosts template file on the template server |
| `template_server.host` | *(required)* | Hostname of the template server (fallback if group is empty) |
| `backup.enabled` | `true` | Whether to create backups before modifying files |

## Behaviour

- The pre-change copy of `/etc/hosts` is written to a fixed path,
  `/etc/hosts.ansible.backup`. It previously carried a timestamp, so every run
  reported `changed` and left another file behind.
- Tasks delegated to the controller run with `become: false`; inheriting the
  play's `become: true` made them attempt `sudo` on the control node.
- `/etc/hosts.ansible.backup` is a snapshot of the file as it was before the
  role first ran; it is written once. Re-copying the live file on every run
  meant the task always reported `changed`. Change-time backups come from the
  merge task's own `backup:`.
- The fetched template is kept at a stable path per host
  (`/tmp/hosts.template.<inventory_hostname>` on the controller and
  `/tmp/hosts.remote` on the target) rather than being written to a timestamped
  name and deleted each run. `fetch` and `copy` compare checksums, so repeat
  runs report no change; deleting the scratch files made every run `changed`.

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL9 System Standardization Configuration
  hosts: rhel9
  become: true
  roles:
    - role: hosts_config
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
