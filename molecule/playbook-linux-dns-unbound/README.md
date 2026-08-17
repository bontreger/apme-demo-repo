# Molecule: `playbook-linux-dns-unbound`

Tests `playbooks/linux-dns-unbound.yml` and the in-repo `unbound_caching_dns` role.

## Run

```bash
export AWS_REGION=us-east-2
molecule test -s playbook-linux-dns-unbound
```

Only `rhel9` is provisioned — playbook OS assert allows 7/8/9 only (RHEL 10 in #60).

## Molecule-specific overrides

| Override | Reason |
|----------|--------|
| `unbound_caching_dns_unbound_allowed_networks` | Stock defaults include `192.168.62.0/24`; molecule VPC is `10.77.0.0/24` |
| `unbound_caching_dns_unbound_forwarder_ip: 8.8.8.8` | Reachable forwarder on EC2 (stock `172.25.254.254` is lab-only) |
| `unbound_caching_dns_unbound_manage_firewall: false` | Cloud AMIs often lack `python3-firewall` |

## AAP

Template: `ACL_linux_dns_unbound_caching`. Inventory group: `caching_dns`. Default EE.
Consider `unbound_caching_dns_unbound_manage_firewall: false` on lab hosts.
