# Introduction

This tool kit is for vm automation management by Ansible on OpenStack, including most common scenarios, including:

1. vm(s) create, delete

2. vm(s) power management, including: start, stop, restart

3. vm(s) resize to different flavor

4. vm(s) snapshot management, including: snapshot create, delete, revert, list

5. vm(s) volume management, including: volume attach, detach, query

6. vm(s) network interface management, including: interface attach, detach

7. vm(s) floating ip management, including: floating ip assign, unassign


The testing platform I used is Red Hat OpenStack Service on OpenShift which can request from Red Hat Demo center:

https://catalog.demo.redhat.com/catalog/babylon-catalog-prod/order/openshift-cnv.osp-on-ocp-cnv.prod)

You should deploy openstack on openshift by following the instruction in showroom at first, then can test all my scenarios.

In this lab, the OpenStack control plane is run on OpenShift as operator, and there is only one compute node with very limited resource(16GB ram and 100GB disk space only), and the compute node is actually a vm, so vms created from the lab will be run in nested virtualization mode. That means I can not install Ansible Automation Platform in this lab, but only install ansible-core. The coming automation scenarios are all done by ansible-playbook command line.

You can run openstack related command from the openstack client pod, e.g：

```
$ oc rsh -n openstack openstackclient
sh-5.1$ openstack compute service list
```


# Prepare the envrionment

Pls prepare below on your bastion server:

1.Install packages:

```
$ sudo yum install ansible-core python3.12-pip -y
$ python3.12 -m pip install --upgrade pip
$ python3.12 -m pip install openstacksdk netaddr
$ python3.12 -c 'import openstack; print("SDK is Ready")'
$ python3.12 -c 'import openstack; print(openstack.__path__)'
```

2.Create the project directory and install required collection in your project directory:

Below is my project directory(~/osp-automation) on bastion server:

```
[lab-user@bastion osp-automation]$ pwd
/home/lab-user/osp-automation

[lab-user@bastion osp-automation]$ ls
ansible.cfg  collections  group_vars  rhel-9.6-x86_64-kvm.qcow2  roles  site.yml  temp_vars.yml

[lab-user@bastion osp-automation]$ cd roles/

[lab-user@bastion roles]$ tree ./
./
└── openstack_vm_manager
    ├── defaults
    │   └── main.yml
    ├── meta
    ├── tasks
    │   ├── config.yml
    │   ├── create.yml
    │   ├── delete.yml
    │   ├── main.yml
    │   ├── migrate.yml
    │   ├── power.yml
    │   ├── setup-flavor.yml
    │   ├── setup-img.yml
    │   └── snapshot.yml
    └── vars

5 directories, 10 files
```

You also need to install collections for coming automation jobs in your project directory:

```
$ ansible-galaxy collection install openstack.cloud -p collections
$ ansible-galaxy collection install community.general -p collections
```

3.Create group_vars/all/vault.yml which save sensitive variables for automation job:

```
$ ansible-vault create group_vars/all/vault.yml
```

The content is like below:

```
vault_os_auth:
  auth_url: "https://keystone-public-openstack.apps.cluster-nrp6v.dynamic.redhatworkshops.io/v3"
  username: "admin"
  password: "openstack"
  project_name: "admin"
  user_domain_name: "Default"
  project_domain_name: "Default"
```

4.Download RHEL 9.x image(here I use RHEL 9.6) and upload it as instance image to your openstack environment:

Please modify your playbook roles/openstack_vm_manager/tasks/setup-img.yml, set path and filename of RHEL 9.x image as your environment.

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=setup-img"
```

5.Modify roles/openstack_vm_manager/tasks/setup-flavor.yml, and run the playbook to set new flavor for your RHEL 9.x instance:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=setup-flavor"
```

and after doing that, you can see new flavor like below:

```
sh-5.1$ openstack flavor list
+--------------------------------------+----------+------+------+-----------+-------+-----------+
| ID                                   | Name     |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+----------+------+------+-----------+-------+-----------+
| 12d2aa90-e41f-42f9-9e2b-147fc6e863c8 | m1.small | 2048 |   20 |         0 |     2 | True      |
| 3ce0e8f6-dc0f-488e-a219-853720b0a8d6 | tiny     |  512 |    1 |         0 |     1 | True      |
+--------------------------------------+----------+------+------+-----------+-------+-----------+
```

# VM Lifecycle automation tasks

The tool kit is actually an ansible role, which contains:

1.site.yml: It's the entry playbook.

```
---
- name: Run OpenStack vm lifecycle automation tool kit
  hosts: localhost
  connection: local
  gather_facts: false

  vars:
    ansible_python_interpreter: /usr/bin/python3.12

  environment:
    PYTHONPATH: "/home/lab-user/.local/lib/python3.12/site-packages"

  roles:
    - role: openstack_vm_manager
tack_vm_manager
```

2.roles/openstack_vm_manager/tasks/<action>.yml: they are playbooks for different operation jobs.

```
[lab-user@bastion roles]$ tree ./
./
└── openstack_vm_manager
    ├── defaults
    │   └── main.yml
    ├── meta
    ├── tasks
    │   ├── config.yml
    │   ├── create.yml
    │   ├── delete.yml
    │   ├── main.yml
    │   ├── migrate.yml
    │   ├── power.yml
    │   ├── setup-flavor.yml
    │   ├── setup-img.yml
    │   └── snapshot.yml
    └── vars
```

3.temp_vars.yml: It's temp variables file for trigger different automation tasks.

### Secnario 1: VM create and delete

1.Edit your temp_vars.yml as below:

```
---
vm_image: "rhel9.6"
vm_flavor: "m1.small"
vm_key_name: "ansible_key"
vm_default_password: "Redhat!234"

vm_list:
  - name: "rhel9-web-node1"
    network: "private"
  - name: "rhel9-web-node2"
    network: "private"
```

2.Run playbook to create vms on openstack according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=create" -e "@temp_vars.yml"
```

3.Run playbook to delete vms on openstack according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=delete" -e "@temp_vars.yml"
```

### Secnario 2: VM power managment

1.Edit your temp_vars.yml as below(can be same as vms create/delete):

```
---
# 镜像与规格的基础定义
vm_image: "rhel9.6"
vm_flavor: "m1.small"
vm_key_name: "ansible_key"
vm_default_password: "Redhat!234" # 使用者期待的默认密码

# 虚拟机实例列表
vm_list:
  - name: "rhel9-web-node1"
    network: "private"
  - name: "rhel9-web-node2"
    network: "private"
```

2.Run playbook to start, stop, restart vms on openstack according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=power" -e "action_type=start" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=power" -e "action_type=stop" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=power" -e "action_type=reboot_soft" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=power" -e "action_type=reboot_hard" -e "@temp_vars.yml"
```

### Secnario 3: VM resize

When handling resize, OpenStack follows the "cold migration" logic by default, that means openstack will check whether there is enough resource for run vms with new bigger flavor, in most situation, it will poweroff the vms and migrate them to new compute node which has enough resource.

But in this testing lab, there is only one compute node, so I didn't run a full test for the scenario, but only give the temp_vars and command for doing this.

1.Edit your temp_vars.yml as below:

```
---
vm_image: "rhel9.6"
new_flavor: "m1.medium"

vm_list:
  - name: "rhel9-web-node1"
  - name: "rhel9-web-node2"
```

2.Run playbook to resize vms on openstack according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=resize" -e "@temp_vars.yml"
```

### Secnario 4: VM snapshot management

1.Edit your temp_vars.yml as below(for vm snapshot create, delete, check, and rollback):

```
---
vm_snapshot_list:
  - vm_name: "rhel9-web-node1"
    snap_name: "rhel9-web-node1-v1"
  - vm_name: "rhel9-web-node2"
    snap_name: "rhel9-web-node2-v1"
```

2.Run playbook for snapshot management on openstack according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=create" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=delete" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=restore" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=query" -e "@temp_vars.yml"
```

### Secnario 5: VM volume management

Prerequisites:

You must have available volumes for attachment before running ansible playbook, you may create new volumes for testing, e.g:

```
$ oc exec -n openstack openstackclient -- openstack volume create --size 5 data-vol-01
$ oc exec -n openstack openstackclient -- openstack volume create --size 5 data-vol-02
$ oc exec -n openstack openstackclient -- openstack volume create --size 5 data-vol-03
$ oc exec -n openstack openstackclient -- openstack volume create --size 5 data-vol-04
```

Then you can verify by running command below:

```
$ oc exec -n openstack openstackclient -- openstack volume list
```

1.Edit your temp_vars.yml as below(for volume attach and detach):

```
---
volume_manage_list:
  - vm_name: "rhel9-web-node1"
    volumes:
      - "data-vol-01"
      - "data-vol-03"
  - vm_name: "rhel9-web-node2"
    volumes:
      - "data-vol-02"
      - "data-vol-04"
```

2.Run playbook for volumes attach, detach or query according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=volume_manage" -e "action_type=attach" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=volume_manage" -e "action_type=detach" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=volume_manage" -e "action_type=query" -e "@temp_vars.yml"
```

### Secnario 6: VM nic management

In Openstack, when attach a private nework nic, the playbook can set fix ip in vm as well, but when attach a public network nic, the playbook can run successfully but no public network ip can see in vm level. As openstack is a public cloud platform, the right way to set a "public network ip" is to assign floating ip instead.

1.Edit your temp_vars.yml as below(for nic attach or detach on specified network):

```
---
interface_manage_list:
  - vm_name: "rhel9-web-node1"
    network: "private"
    fixed_ip: "10.0.0.11"
  - vm_name: "rhel9-web-node2"
    network: "private"
    fixed_ip: "10.0.0.12"
```

2.Run playbook for nic attach or detach according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=interface_manage" -e "action_type=attach" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=interface_manage" -e "action_type=detach" -e "@temp_vars.yml"
```

### Secnario 7: VM floating ip management

Prerequisites:

You must have available floating ips for assignment before running ansible playbook, you may check by running:

```
$ oc exec -n openstack openstackclient -- openstack floating ip list
```

1.Edit your temp_vars.yml as below(for floating ip assign or unassign):

```
---
floating_ip_list:
  - vm_name: "rhel9-web-node1"
    fip_address: "192.168.6.55"
  - vm_name: "rhel9-web-node2"
    fip_address: "192.168.6.56"
```

2.Run playbook for floating ip assign or unassign according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=fip_manage" -e "action_type=assign" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=fip_manage" -e "action_type=unassign" -e "@temp_vars.yml
```

### Secnario 8: Security Group management

Prerequisites:

You must have available security groups for assignment before running ansible playbook, you may check by running:

```
$ oc exec -n openstack openstackclient -- openstack security group lis
```

1.Edit your temp_vars.yml as below(for changing vms to attach specified security group):

```
---
  - vm_name: "rhel9-web-node1"
    sg_name: "test-security-group"
  - vm_name: "rhel9-web-node2"
    sg_name: "test-security-group"
```

2.Run playbook for attach or query security group on specified vms according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=secgroup_manage" -e "action_type=attach" -e "@temp_vars.yml"
$ ansible-playbook site.yml --ask-vault-pass -e "operation=secgroup_manage" -e "action_type=query" -e "@temp_vars.yml"
```

### Secnario 9: Check or Inspect vms information

1.Edit your temp_vars.yml as below(to specified vms):

2.Run playbook for checking info on specified vms according to your variables:

```
$ ansible-playbook site.yml --ask-vault-pass -e "operation=inspection" -e "@temp_vars.yml"
```
