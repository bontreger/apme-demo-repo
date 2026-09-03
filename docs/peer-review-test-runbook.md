# Playbook test & validate runbook (repeatable)

High-level process used for **`linux-nfs-server.yml`** (#23). Use the same flow for other peer-review playbooks.

---

## Overview — three phases

| Phase | What it proves | Where it runs |
|-------|----------------|---------------|
| **1. Molecule** | Playbook works on a real OS; automated verify | EC2 controller → temp test VM(s) in AWS |
| **2. GPA** | Code quality & good practices | Your Mac (ansible-lint + manual read) |
| **3. AAP** | Works via Automation Platform (job template, inventory, creds) | Team/demo AAP → your lab RHEL host |

**Issue hygiene:** One peer-review issue per playbook → comment results → close; open **new issues** for defects or **umbrella issues** for repo-wide gaps (e.g. RHEL 10).

---

## Before you start

- [ ] Peer-review issue assigned to you (e.g. `Peer review: playbooks/linux-*.yml`)
- [ ] Local clone of `acl-ready-to-use-content` (latest `main`)
- [ ] AWS sandbox access (for Molecule + lab target)
- [ ] AAP URL + login (teammate or lab)
- [ ] GitHub PAT with `repo` scope (private repo sync in AAP)

---

## Phase 1 — Molecule (lab execution test)

### 1.1 Provision EC2 **controller** (one-time per tester)

1. AWS Console → **EC2** → Launch instance
2. **AMI:** Amazon Linux 2023 or RHEL (controller only)
3. **Type:** `t3.medium`+
4. Key pair → save `.pem`
5. Default VPC, **public IP**, SSH (22) from your IP
6. Attach **IAM role** with EC2 permissions (or use access keys)
7. SSH: `ssh -i key.pem ec2-user@<controller-ip>`

### 1.2 Install tooling on controller

```bash
sudo dnf install -y python3.11 python3.11-pip git
python3.11 -m pip install --user 'molecule>=25.7.0' boto3 pywinrm
export PATH="$HOME/.local/bin:$PATH"
ansible-galaxy collection install -r molecule/requirements.yml
ansible-galaxy collection install -r collections/requirements.yml
export AWS_REGION=us-east-2   # or your region
```

**Note:** Molecule 26+ needs **Python 3.10+** (not 3.9 on AL2023 default).

### 1.3 Get repo onto controller

```bash
# From Mac — tarball often easiest
tar czf /tmp/acl-ready-to-use-content.tar.gz acl-ready-to-use-content
scp -i key.pem /tmp/acl-ready-to-use-content.tar.gz ec2-user@<controller-ip>:~/
# On controller
tar xzf acl-ready-to-use-content.tar.gz && cd acl-ready-to-use-content
```

Include any **new Molecule scenario** if not yet on `main`.

### 1.4 Scenario naming & OS

- Scenario: `playbook-<playbook-name-without-.yml>`
  e.g. `playbook-linux-nfs-server` → `playbooks/linux-nfs-server.yml`
- Check playbook `hosts:` and OS assert — map inventory groups (e.g. `all_rhel`)
- If OS assert is 7/8/9 only, use **`rhel9` only** in scenario inventory (not `rhel10`)

### 1.5 Run test

```bash
cd ~/acl-ready-to-use-content
export AWS_REGION=us-east-2
export PATH="$HOME/.local/bin:$PATH"
molecule test -s playbook-<name>
```

**Pass:** syntax, create, converge, verify, destroy all successful.

### 1.6 Record on issue

- Scenario name, OS, region, pass/fail, notes
- Terminate controller EC2 when done (optional; keep for more tests if useful)

---

## Phase 2 — GPA review

### 2.1 Automated lint (Mac or CI)

```bash
cd acl-ready-to-use-content
ansible-galaxy collection install -r collections/requirements.yml
ansible-lint playbooks/<playbook>.yml roles/<role>/
```

**Pass:** `0 failure(s), 0 warning(s)` (production profile).

### 2.2 Manual review ([GPA](https://redhat-cop.github.io/automation-good-practices/))

Read playbook + role(s):

- Thin playbook, logic in roles
- FQCN, prefixed variables, README
- Idempotency, validation, error handling
- Tags, firewalld/service edge cases
- OS support vs docs/Molecule (`RHEL 10` gaps, etc.)

**Outcome:** Pass / pass with recommendations / fail → fix issue if blocking.

### 2.3 Record on issue

- ansible-lint result
- Short manual GPA notes
- Link **umbrella/follow-up issues** for non-blocking repo-wide items (don't block close)

---

## Phase 3 — AAP test

### 3.1 Project (once per AAP, or if missing)

1. **Credentials** → Source Control: `x-access-token` + GitHub PAT (`repo`, SSO authorized)
2. **Projects** → `acl-ready-to-use-content`
   - HTTPS URL, branch `main`, SCM credential
3. **Sync** → Successful

### 3.2 Lab target RHEL host

1. AWS EC2 → **RHEL 8 or 9** (match playbook OS assert; not RHEL 10 if assert excludes it)
2. Official Red Hat AMI, **x86_64**, `t3.small`, public IP, SSH open
3. SSH from Mac with same key used at launch: `ec2-user@<ip>`

### 3.3 Inventory

1. **Inventories** → lab inventory
2. Host (e.g. `rhel9_lab`) **Variables:**

```yaml
ansible_host: <EC2_PUBLIC_IP>
ansible_user: ec2-user
```

3. **Group** matching playbook `hosts:` (e.g. **`all_rhel`**) → add host

### 3.4 Machine credential

- **Type:** Machine
- **Name:** e.g. `acl-linux-lab`
- **Username:** `ec2-user`
- **SSH private key:** paste `.pem` used at EC2 launch

(Demo Credential only works for pre-configured demo nodes, not your EC2.)

### 3.5 Job template

| Field | Value |
|-------|--------|
| Name | `ACL_linux_<use_case>` (see `controller_config/job_templates.yml`) |
| Project | `acl-ready-to-use-content` |
| Playbook | `playbooks/<playbook>.yml` |
| Inventory | Lab inventory |
| EE | Default execution environment |
| Credential | `acl-linux-lab` |

Or run team **CaC** (`controller_config/configure_aap.yml`) if templates already exist.

### 3.6 Launch

- **Limit:** your host name (optional)
- **Extra vars:** role-specific (e.g. NFS export CIDR)
- Wait for **Successful**
- If **UNREACHABLE:** widen EC2 SG SSH (AAP source ≠ your Mac IP)

### 3.7 Record on issue

- Job template name, job ID/link, target OS, pass/fail

---

## Close peer-review issue

Per issue template:

1. Comment: Molecule + GPA + AAP results
2. **Pass** → close issue
3. **Defect** → new fix issue, link, then close peer review
4. **Repo-wide gap** (e.g. RHEL 10) → umbrella issue, link, close peer review

---

## Quick checklist (per playbook)

```
[ ] Issue assigned
[ ] Molecule scenario exists / created
[ ] molecule test -s playbook-<name>  → PASS
[ ] ansible-lint playbook + role(s)   → PASS
[ ] Manual GPA review                 → PASS (notes optional)
[ ] AAP: host in correct group + machine cred
[ ] AAP: job template launch          → Successful
[ ] Results commented on issue
[ ] Follow-up issues filed if needed
[ ] Issue closed
```

---

## Common pitfalls (from NFS peer review)

| Pitfall | Fix |
|---------|-----|
| Molecule config error (`ansible` unexpected) | Upgrade to Molecule 26 + Python 3.11 |
| `pip install molecule` only gets 6.x on PyPI for Py 3.9 | Use Python 3.11 |
| EC2 IAM can't create test VMs | Attach EC2-capable IAM role to controller |
| Wrong AMI (SQL bundle, ARM) | Red Hat RHEL 8/9 x86_64 HVM |
| AAP project sync fails | HTTPS URL + PAT, SSO authorize org |
| AAP job UNREACHABLE | SG allow SSH from AAP (not just My IP) |
| OS assert fails on RHEL 10 | Use RHEL 8/9 for test; track RHEL 10 in umbrella issue |
| `service_facts` says nfs-server not running | Use `systemd` ActiveState `active` in verify (oneshot units) |

---

## What varies per playbook

| Item | Where to look |
|------|----------------|
| Inventory group | Playbook `hosts:` line |
| Extra vars at launch | Role `defaults/main.yml` + README |
| Molecule scenario inventory | Map shared hosts into playbook groups |
| OS versions allowed | Playbook/role `assert` pre_task |
| Job template name | `controller_config/job_templates.yml` |

---

## Related docs

- [getting-started.md](getting-started.md)
- [aap-deployment-guide.md](aap-deployment-guide.md)
- [molecule/README.md](../molecule/README.md)
- [docs/molecule/adding-scenarios.md](molecule/adding-scenarios.md)
