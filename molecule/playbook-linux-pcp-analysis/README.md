# playbook-linux-pcp-analysis scenario

Tests [`playbooks/linux-pcp-analysis.yml`](../../playbooks/linux-pcp-analysis.yml)
(install on `all_rhel`, collect and write reports on `pcp_servers`).

Provisions **rhel9 and rhel10** from the shared linux inventory. Expect a
longer converge (PCP package install + collection).

## Running

```bash
source hack/molecule-iam.env
molecule test -s playbook-linux-pcp-analysis
```
