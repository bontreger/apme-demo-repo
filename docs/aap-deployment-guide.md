# AAP Deployment Guide

Deploy ACL ready-to-implement content into Ansible Automation Platform (AAP) 2.5+
using Configuration as Code.

---

## Prerequisites

### 1. AAP access

You need a running AAP 2.5+ controller and a **Personal Access Token** (PAT)
with permission to create projects, inventories, and job templates.

To create a PAT: **AAP UI > your username (top-right) > Personal access tokens
> Create personal access token**. Set the scope to **Write**.

### 2. Credentials in AAP

The CaC playbook does **not** create credentials -- you must create them first.

| Credential | Type | Purpose |
|------------|------|---------|
| Linux machine credential | Machine | SSH access to RHEL targets |
| Windows machine credential | Machine | WinRM access to Windows targets |
| SCM credential (if private repo) | Source Control | Git clone access for AAP project sync |

Note the **exact names** you give these credentials; you will enter them in
`site_config.yml`.

### 3. Install the CaC collection

On the workstation where you will run the CaC playbook:

```bash
ansible-galaxy collection install infra.aap_configuration
```

`infra.aap_configuration` internally uses `ansible.controller` modules.
`ansible.controller` is a Red Hat subscription-gated collection and is **not
available on the public Ansible Galaxy**. Use the open-source equivalent
`awx.awx` instead and create a namespace symlink so Ansible resolves it
correctly:

```bash
# Install the open-source equivalent (pin to 23.x for compatibility)
ansible-galaxy collection install 'awx.awx:==23.9.0'

# Create a namespace symlink so ansible.controller resolves to awx.awx
mkdir -p collections/ansible_collections/ansible
ln -s ../awx/awx collections/ansible_collections/ansible/controller
```

> **Note:** If you have a Red Hat Automation Hub subscription, you can install
> `ansible.controller` directly from Hub and skip the symlink step.

### 4. Clone the repository

```bash
git clone https://github.com/redhat-services-platform-team/acl-ready-to-implement-content.git
cd acl-ready-to-implement-content
```

---

## Step-by-step deployment

### Step 1 -- Create site configuration

```bash
cp controller_config/site_config.yml.example controller_config/site_config.yml
```

Edit `controller_config/site_config.yml` and fill in your values:

```yaml
# AAP connection
aap_hostname: https://your-aap-host.example.com
aap_validate_certs: false          # true if using a trusted certificate

# Project -- Git URL of this repository as AAP will clone it
acl_project_scm_url: https://github.com/your-org/acl-ready-to-implement-content.git
acl_project_scm_branch: main
acl_project_scm_credential: ""     # name of SCM credential, or empty for public repos

# Credentials -- must match names you created in AAP
acl_linux_credential: my_linux_cred
acl_windows_credential: my_windows_cred

# Organization and inventory
acl_organization: Default
acl_inventory_name: acl_lab_inventory
```

`site_config.yml` is gitignored and will not be committed.

### Step 2 -- Run the CaC playbook

```bash
ansible-playbook controller_config/configure_aap.yml \
  -e @controller_config/site_config.yml \
  -e aap_token=YOUR_PAT_HERE
```

The playbook validates required variables, then creates (or updates) the
following AAP objects in order:

1. **Inventory** -- `acl_lab_inventory` (or the name you chose)
2. **Project** -- `acl-ready-to-implement-content`, pointing to your Git URL
3. **Job templates** -- 40 templates (35 Linux, 5 Windows)

### Step 3 -- Add hosts to inventory

The CaC playbook creates an empty inventory. Add hosts via the AAP UI or API:

```
AAP UI > Inventories > acl_lab_inventory > Hosts > Add
```

Playbooks target inventory groups directly -- hosts must belong to the
correct group or they will not be matched. Use AAP's **limit** field at
launch time to narrow down to specific hosts within a group.

**Required groups** (most playbooks need these):

| Group | Used by |
|-------|---------|
| `all_rhel` | Most Linux playbooks (RCA, inspection, config, security) |
| `all_windows` | All Windows playbooks (service, IIS, patching, software) |

**Specialized groups** (only needed if you use the corresponding playbooks):

| Group | Playbook(s) |
|-------|-------------|
| `primary_dns` | `linux-dns-primary.yml` |
| `secondary_dns` | `linux-dns-secondary.yml` |
| `caching_dns` | `linux-dns-unbound.yml` |
| `dhcp_servers` | `linux-dhcp-server.yml` |
| `dhcp_clients` | `linux-dhcp-client.yml` |
| `postgres_servers` | `linux-postgresql-deploy.yml` |
| `pcp_servers` | `linux-pcp-analysis.yml` (second play) |

A host can belong to multiple groups (e.g., a RHEL host in both `all_rhel`
and `postgres_servers`).

### Step 4 -- Verify project sync

After the CaC playbook runs, check that the project synced successfully:

```
AAP UI > Projects > acl-ready-to-implement-content > Status
```

If the status shows **Failed**, common causes:

- **Private repository without SCM credential** -- create an SCM credential
  in AAP and set `acl_project_scm_credential` in `site_config.yml`, then
  re-run the CaC playbook.
- **Branch does not exist** -- verify `acl_project_scm_branch`.
- **Network connectivity** -- ensure AAP can reach the Git host.

### Step 5 -- Run a test job

Launch a low-risk job template to validate the setup:

```
AAP UI > Templates > ACL_linux_os_info_report > Launch
```

This collects OS facts and generates a report without making changes.

---

## What gets created

| AAP object | Count | Details |
|------------|:-----:|---------|
| Inventory | 1 | Empty shell; add hosts manually |
| Project | 1 | Git repo with `scm_update_on_launch: true` |
| Job templates -- Linux | 35 | DNS, DHCP, database, config, inspection, RCA, security |
| Job templates -- Windows | 5 | Service mgmt, IIS, software, Chocolatey, patching |

All job templates reference the project `acl-ready-to-implement-content` and use
the `Default execution environment`. Credential and inventory names are
derived from `site_config.yml`.

---

## Re-running the playbook

The CaC playbook is **idempotent**. Running it again will:

- Update existing objects if the YAML definitions changed.
- Skip objects that already match.
- **Not** delete objects that were removed from YAML (deletion is not managed).

This means you can safely re-run after changing `site_config.yml` or after
updating the repository with new playbooks.

---

## Deploying a selected subset of templates (optional)

By default, `configure_aap.yml` deploys all available job templates.
If a customer engagement only requires a specific set of playbooks, you can
limit deployment to just those templates by adding a `selected_playbooks` list
to `site_config.yml`.

### Option A — Edit site_config.yml directly

Add the following to your `site_config.yml` (the key is absent by default):

```yaml
selected_playbooks:
  - playbooks/linux-dns-primary.yml
  - playbooks/linux-firewall-management.yml
  - playbooks/linux-os-info-report.yml
```

Then run the CaC playbook as normal:

```bash
ansible-playbook controller_config/configure_aap.yml \
  -e @controller_config/site_config.yml \
  -e aap_token=YOUR_PAT_HERE
```

Only the listed templates will be created in AAP. All other CaC objects
(inventory, project) are always deployed regardless of this setting.

### Option B — Use the ACL Library Deployment Package Builder

The [Ansible Content Library](https://script.google.com/macros/s/AKfycbxYi897VljbD7frbdwr2sRtgIvq_73Mn_bNFSmtmvrR7mY07EyreCNf_TaH7_V5V-ythw/exec)
includes a **Deployment Package Builder** that lets you browse and filter all
available templates, check the ones relevant to the engagement, and generate a
ready-to-use `site_config.yml` with the `selected_playbooks` list pre-populated.

> **Note:** If `selected_playbooks` is absent or an empty list, all templates
> are deployed — identical behaviour to before this feature was added.

---

## Troubleshooting

### "Project update failed"

The project was created but AAP could not clone the repository.

1. Check if the repo is private -- you need an SCM credential.
2. Verify the URL in `acl_project_scm_url` is reachable from the AAP node.
3. Check AAP project activity logs for the specific Git error.

### "Credential not found" during job template creation

The credential name in `site_config.yml` does not match any credential in AAP.

1. Open AAP UI > Credentials and verify the exact name.
2. Update `acl_linux_credential` or `acl_windows_credential` in
   `site_config.yml` to match.

### "Missing required variables" assertion failure

You forgot to pass `site_config.yml` or `aap_token`.

```bash
ansible-playbook controller_config/configure_aap.yml \
  -e @controller_config/site_config.yml \
  -e aap_token=YOUR_PAT
```

### Job template launches fail with "No hosts matched"

The inventory is empty. Add hosts via AAP UI or API (Step 3 above).

### Windows templates fail with WinRM errors

Verify:

1. WinRM is enabled on the Windows target (`winrm quickconfig`).
2. The Windows machine credential has the correct username/password.
3. AAP can reach the target on port 5985 or 5986.
