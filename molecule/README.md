# Molecule testing

Self-contained molecule test infrastructure on AWS EC2 — no external
provisioner collection required. Targets RHEL 9/10 (SSH) by default; Windows
Server (WinRM over HTTPS, official Amazon-published AMIs) is opt-in per
scenario, so `win_*` content is testable the same way as Linux content.

## Layout

```
.config/molecule/config.yml # Shared base config, deep-merged under every
                            # scenario's molecule.yml (scenario keys win)
molecule/
├── default/                # Smoke-test scenario — see its README.md
├── playbook-linux-yum-repo-config/   # example: tests playbooks/linux-yum-repo-config.yml
├── role-postgresql-server/           # example: tests roles/postgresql_server
├── playbook-linux-postgresql-deploy/  # tests playbooks/linux-postgresql-deploy.yml
├── playbook-win-chocolatey/           # tests playbooks/win-chocolatey.yml
├── playbook-win-iis-deploy/           # tests playbooks/win-iis-deploy.yml
├── playbook-win-patch-management/     # tests playbooks/win-patch-management.yml
├── playbook-win-service-management/   # tests playbooks/win-service-management.yml
├── playbook-win-software-management/  # tests playbooks/win-software-management.yml
├── utils/                  # Shared test infrastructure (not a scenario)
│   ├── inventory/          # Shared inventory layers
│   │   ├── base/           # localhost + molecule_aws_* group_vars (always loaded)
│   │   ├── linux/          # rhel10, rhel9 (loaded by default)
│   │   └── windows/        # win2022 (opt-in)
│   └── provisioners/
│       └── aws/            # EC2: throwaway VPC + key pair + SG + instance
│           ├── create.yml  # also waits for guest connectivity at the end
│           └── destroy.yml
├── requirements.txt        # python deps (molecule, boto3, pywinrm)
└── requirements.yml        # collection deps (amazon.aws, community.aws, community.crypto)
```

The `dependency` step also installs the repo's `collections/requirements.yml`
(`ansible.windows`, `community.windows`, `chocolatey.chocolatey`,
`community.postgresql`, ...) — the collections the *content* under test needs,
as opposed to `molecule/requirements.yml` which covers the test infrastructure
itself. Both go to the galaxy default path, not the repo's `collections/` tree:
molecule generates its own `ansible.cfg`, and a converge playbook's
`playbook_dir` is the scenario directory, so nothing would discover
`<repo>/collections`.

Content scenarios follow a naming convention: a scenario testing a playbook is
named `playbook-<playbook-name-without-file-extension>` (e.g.
`playbook-linux-yum-repo-config` for `playbooks/linux-yum-repo-config.yml`),
and a scenario testing a role is named `role-<name-of-role>` (e.g.
`role-postgresql-server` for `roles/postgresql_server`).

The base config wires `create`/`destroy` to `../utils/provisioners/aws/`, `prepare`
to an optional scenario-local `prepare.yml` (skipped when absent), the
test sequence, `ANSIBLE_ROLES_PATH` so converge playbooks resolve the repo's
`roles/` by name, and the default inventory stack: `../utils/inventory/base/` +
`../utils/inventory/linux/` plus an optional scenario-local `inventory/` (later
sources win). The runtime inventory — connection details the create playbook
writes into the molecule ephemeral directory — is loaded by molecule itself
on every command and is not listed in the stack.
The utils provision exactly the hosts the loaded inventories declare. A new
scenario is just a directory with an (empty) `molecule.yml`, `converge.yml`,
and `verify.yml`.

## Per-host variables

These are resolved per host by the provisioner playbooks, so one scenario can
mix operating systems — `utils/inventory/linux/` ships `rhel10` and `rhel9`,
`utils/inventory/windows/` ships `win2022`. Set them on host entries in the
inventory's `hosts.yml` (shared or scenario-local); everything else (region,
ssh user, VPC settings, ...) is group-wide in
`utils/inventory/base/group_vars/all.yml`.

| Variable                     | Purpose                                                                                                  | Default                    |
| ---------------------------- | -------------------------------------------------------------------------------------------------------- | -------------------------- |
| `molecule_aws_image_id`      | Exact AMI ID; takes precedence — the name lookup is skipped for that host                                | unset                      |
| `molecule_aws_ami_name`      | AMI name filter; the newest match wins                                                                   | per host (see `hosts.yml`) |
| `molecule_aws_ami_owner`     | AMI owner account the lookup is scoped to                                                                | `309956199498` (Red Hat)   |
| `molecule_aws_connection`    | Guest connection: `ssh` or `winrm` (Windows — HTTPS 5986, Administrator password via `ec2_win_password`) | `ssh`                      |
| `molecule_aws_instance_type` | EC2 instance type                                                                                        | `t3.medium`                |

Scenarios can layer additional per-host vars for their own converge/verify via
a scenario-local `inventory/` — the default scenario's
[README](default/README.md) shows the pattern.

Group-wide values are overridden the same way, in a scenario-local
`inventory/group_vars/all.yml`. `molecule_aws_extra_ports` (default `[]`) is
the one most scenarios reach for: inbound TCP ports added to the test security
group on top of 22 and 5986, needed when the content serves traffic — see
[playbook-win-iis-deploy](playbook-win-iis-deploy/inventory/group_vars/all.yml).
It has to be group-wide because `create.yml` reads it on `localhost`.

## Adding a scenario

A scenario is a directory under `molecule/` with three files; everything else
is inherited from the base config and shared inventory. Name it per the
convention above and run it with `molecule test -s <name>`.

By default a scenario gets the Linux stack (`base/` + `linux/`). Windows is
opt-in: declare the stack in the scenario's `molecule.yml` with the windows
layer, as `molecule/default/` does (the args list replaces the base config's
wholesale — drop the linux line for Windows-only):

```yaml
---
ansible:
  executor:
    args:
      ansible_playbook:
        - --inventory=../utils/inventory/base/
        - --inventory=../utils/inventory/linux/
        - --inventory=../utils/inventory/windows/
        - --inventory=inventory/
```

### Testing Windows content

The five `playbook-win-*` scenarios are the worked examples: each drops the
linux layer (Windows-only, so only `win2022` gets provisioned), maps `win2022`
into the `all_windows` group the `win-*` playbooks target, and supplies the
role vars the content needs — every `win_*` role defaults to an empty
package/service list, so a converge on defaults would be a no-op. Notes worth
carrying into new Windows scenarios:

- Do **not** add `become: true` the way the Linux role examples do. WinRM
  already connects as Administrator, and `become` on Windows is a different
  mechanism (`runas`).
- Verify from inside the guest (`win_uri` against `http://localhost/`) rather
  than from the controller, so assertions don't depend on the security group.
- `ansible.windows.win_updates` drives Windows Update through a Task Scheduler
  job, so it works over the runtime inventory's NTLM connection without
  `become`.
- Windows first boot is slow — EC2Launch, the WinRM `user_data`, then waiting
  for the Administrator password. `create.yml` allows 900s for the connection
  (vs 300s for Linux); budget several minutes before converge starts.
- `playbook-win-software-management` pins an external MSI URL plus its exact
  `product_id`; both have to be updated together when that version disappears.

### Testing a playbook

Scenario name: `playbook-<playbook-name-without-file-extension>`.

```bash
mkdir -p molecule/playbook-linux-yum-repo-config/inventory
printf -- '---\n' > molecule/playbook-linux-yum-repo-config/molecule.yml
```

`converge.yml` imports the playbook under test:

```yaml
---
- name: Converge
  ansible.builtin.import_playbook: ../../playbooks/linux-yum-repo-config.yml
```

Repo playbooks target their own groups (this one has `hosts: server` and
`hosts: client` plays), so map the test instances into those groups with a
scenario-local `inventory/hosts.yml` — membership is additive, the hosts stay
in `molecule` for the lifecycle playbooks:

```yaml
---
all:
  children:
    server:
      hosts:
        rhel10:
    client:
      hosts:
        rhel9:
```

`verify.yml` asserts the playbook's outcomes (services running, reports
generated, ...) against `hosts: molecule`.

### Testing a role

Scenario name: `role-<name-of-role>`.

```bash
mkdir molecule/role-postgresql-server
printf -- '---\n' > molecule/role-postgresql-server/molecule.yml
```

`converge.yml` applies the role directly — repo roles resolve by name via the
`ANSIBLE_ROLES_PATH` the base config sets:

```yaml
---
- name: Converge
  hosts: molecule
  become: true
  roles:
    - role: postgresql_server
```

Role vars go under the play's `vars:`, or in a scenario-local
`inventory/hosts.yml` (per host) / `inventory/group_vars/` when they differ
per instance. `verify.yml` asserts the role's outcomes.

Verbose walkthroughs — full file contents, verify examples, molecule.yml
override cookbook, and how base-config/inventory inheritance works — are in
[docs/molecule/](../docs/molecule/).

## Setup

Molecule itself must be installed (`pip install molecule`); everything else —
python deps and collections — is installed automatically by the `dependency`
step on every run (a ~1s no-op once satisfied). To pre-install manually:

```bash
pip install -r molecule/requirements.txt
ansible-galaxy collection install -r molecule/requirements.yml
```

## Running

```bash
# Needs AWS credentials in the environment (AWS_ACCESS_KEY_ID/
# AWS_SECRET_ACCESS_KEY or AWS_PROFILE) and optionally AWS_REGION.
# With privileged sandbox credentials you can instead mint a minimally
# scoped IAM user dedicated to these tests — see
# docs/molecule/aws-iam-user.md for its exact permissions and teardown:
#   ansible-playbook hack/create-molecule-iam-user.yml
#   source hack/molecule-iam.env
molecule test

# Additional scenarios: molecule test -s <scenario>, or --all for everything.
```

## AWS networking

By default create.yml provisions a throwaway VPC, public subnet, internet
gateway, and route table named `molecule-<scenario>` and launches instances
there; destroy.yml tears it all down. No default VPC or pre-existing
networking is required. The CIDR is `molecule_aws_vpc_cidr` (default
`10.77.0.0/24`).

To reuse existing vpc/subnets:

```bash
export MOLECULE_AWS_VPC_ID=vpc-0123456789abcdef0
export MOLECULE_AWS_SUBNET_ID=subnet-0123456789abcdef0
molecule test
```
