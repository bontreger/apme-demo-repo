# Molecule testing

Self-contained molecule test infrastructure — no external provisioner
collection required. RHEL 9 and RHEL 10 targets.

## Layout

```
.config/molecule/config.yml # Shared base config, deep-merged under every
                            # scenario's molecule.yml (scenario keys win)
molecule/
├── utils/                  # Shared lifecycle playbooks (not a scenario)
│   ├── aws/                # EC2: throwaway VPC + key pair + SG + instance
│   │   ├── create.yml
│   │   ├── destroy.yml
│   │   └── prepare.yml
│   └── kubevirt/           # VM cloned from a DataSource + NodePort SSH
│       ├── create.yml
│       ├── destroy.yml
│       └── prepare.yml
├── inventory/              # Shared inventory used by every scenario
│   ├── hosts.yml           # `molecule` group = test instances (rhel10, rhel9)
│   └── group_vars/all.yml  # molecule_aws_* / molecule_kubevirt_* defaults
├── default/                # Example scenario — just the test itself
│   ├── molecule.yml        # empty; overrides only
│   ├── converge.yml
│   ├── verify.yml
│   └── inventory/          # scenario-local vars layered over the shared
│       └── hosts.yml       # inventory (molecule_expected_rhel_major)
├── requirements.txt        # python deps (molecule, boto3, kubernetes)
└── requirements.yml        # collection deps (amazon.aws, kubernetes.core, community.crypto)
```

The base config wires `create`/`destroy`/`prepare` to
`../utils/${PROVISIONER:-aws}/`, the test sequence, and a three-layer
inventory stack: shared (`../inventory/`), optional scenario-local
(`inventory/`, later sources win), and the runtime inventory the create
playbooks write SSH connection details to in the molecule ephemeral
directory. A new scenario is just a directory with an (empty) `molecule.yml`,
`converge.yml`, and `verify.yml`.

## Per-host images

Image selection is resolved per host, so one scenario can mix RHEL versions —
the shared inventory ships `rhel10` and `rhel9` as an example. Each host
defines its image identity on its entry in `hosts.yml`:

- `molecule_aws_ami_name` — AMI lookup filter (newest match wins);
  `molecule_aws_image_id` pins an AMI and skips the lookup.
  `molecule_aws_ami_owner` defaults to Red Hat's publisher account in
  `group_vars/all.yml` and can also be set per host.
- `molecule_kubevirt_data_source_name` — the DataSource the root disk is
  cloned from.

## Setup

```bash
pip install -r molecule/requirements.txt
ansible-galaxy collection install -r molecule/requirements.yml
```

## Running

```bash
# AWS (default) — needs AWS credentials in the environment (AWS_ACCESS_KEY_ID/
# AWS_SECRET_ACCESS_KEY or AWS_PROFILE) and optionally AWS_REGION.
molecule test

# KubeVirt — needs KUBECONFIG pointing at a cluster with KubeVirt/OpenShift
# Virtualization and rhel9/rhel10 DataSources.
PROVISIONER=kubevirt molecule test

# Additional scenarios: molecule test -s <scenario>, or --all for everything.
```

Per-platform settings (region, instance type, namespace, DataSource, disk
size, ...) live in each scenario's `inventory/group_vars/all.yml` as
`molecule_aws_*` / `molecule_kubevirt_*` vars.

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
