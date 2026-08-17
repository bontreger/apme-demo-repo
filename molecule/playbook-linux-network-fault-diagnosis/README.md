# playbook-linux-network-fault-diagnosis scenario

Tests [`playbooks/linux-network-fault-diagnosis.yml`](../../playbooks/linux-network-fault-diagnosis.yml)
(four roles: `network_interface`, `system_diagnosis`, `traffic_capture`,
`report_generation`). Converge runs the full diagnosis playbook on a stock RHEL
9 EC2 instance; `verify.yml` asserts the plaintext report was written.

## rhel9 only

Drops the shared linux layer and provisions only `rhel9`. The playbook's
`pre_tasks` assert `ansible_distribution` in `RedHat`/`CentOS` and major
version `7`/`8`/`9`.

## Molecule overrides

| Variable | Value | Why |
|----------|-------|-----|
| `network_interface` | `eth0` | AWS EC2 primary NIC (role default `ens192` is wrong here) |
| `report_output_dir` | `/var/lib/ansible-reports/network_diagnosis` | Required by playbook pre_task assert |
| `report_job_id` | `molecule` | Deterministic report path for verify |
| `traffic_capture.enabled` | `false` | Skips ~30s tcpdump (report still generated) |

## AAP

Template: `ACL_linux_network_fault_diagnosis`. Inventory: `all_rhel`. Suggested
extra vars for lab:

```yaml
network_interface: eth0
traffic_capture:
  enabled: false
```

## Running

```bash
molecule test -s playbook-linux-network-fault-diagnosis
```

```bash
molecule syntax -s playbook-linux-network-fault-diagnosis
ansible-lint molecule/playbook-linux-network-fault-diagnosis/
```
