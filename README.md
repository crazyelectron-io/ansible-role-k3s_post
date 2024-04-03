# ansible-role-k3s_post

Ansible role to setup MetalLB after K3s cluster deployment.

> Be aware that this role is highly opinionated and fits my preferences and way of working. It may or may not be suitable for your needs.

The hosts to target must be in a group named `master`, e.g.:

```yaml
# file: inventories/dev/hosts.yml
# synopsis: inventory file for development environment
---
all:
  children:
    master:
      hosts:
        kube-m01:
          ansible_host: 10.100.0.11
        kube-m02:
          ansible_host: 10.100.0.12
```

## Role variables

Variables can be specified in the `hosts.yaml` inventory file, in the `group_vars` directory, or in the `vars` section of an Ansible playbook, like this:

```yaml
- hosts: all
  become: true
  gather_facts: true
  vars:
    metal_lb_ip_range: "10.20.30.40-10.20.30.50"
    apiserver_endpoint: "10.10.10.10"
    k3s_token: "th1sisaSupesecretToken"
  roles:
    ...
    - k3s-post
    ...
```

An `hosts.yaml` inventory file will look like this:

```yaml
---
all:
  vars:
    - metal_lb_ip_range: "10.20.30.40-10.20.30.50"
    - apiserver_endpoint: "10.10.10.10"
    ...
  children:
  ...
```

### Mandatory variables

None.

### Optional variables

`metal_lb_available_timeout` - Can define a different timeout value for your environment; default is 120 seconds.

### Optional parameters (with defaults)

`metal_lb_available_timeout` - Timeout to wait for MetalLB services to come up.

## Usage of this role

To use this role, include the following section in a requirements.yml file in the local roles directory:

```ini
# Include the 'debian-base` role from GitHub
- src: git@github.com:crazyelectron-io/role-k3s_master.git
  scm: git
  version: main
  name: k3s-master
```

Only include the 'top' roles, dependencies - when listed in meta/main.yml of the imported role - will be downloaded automatically.

To retrieve roles like this in your project, run ansible-galaxy install -r roles/requirements.yml. Because these roles will not be updated locally when the repository is changed, to refresh an already retrieved role use ansible-galaxy install -f -r roles/requirements.yml

A playbook to use this role would look like this:

```yaml
- hosts: master
  become: true
  gather_facts: true
  roles:
    - role: k3s-master
      when: install_k3s | bool
```

### Dependencies

None.

### Suggested project structure

```shell
├── inventories
│   ├── dev
│   │   ├── group_vars/
│   │   └── hosts.yml
│   └── prod
│       ├── group_vars/
│       └── hosts.yml
├── group_vars/
├── host_vars/
├── files/
├── templates/
├── roles
│   ├── local
│   │   ├── local_role1/
│   │   └── local_role2/
│   ├── requirements.yml
│   ├── .gitignore
├── ansible.cfg
├── README.md
├── some_playbook.yml
├── other_playbook.yml
```

Create a `roles/.gitignore` file to exclude the downloaded roles:

```shell
#Ignore everything in roles dir...
/*
# ... but current file...
!.gitignore
# ... external role requirement file
!requirements.yml
# ... and configured custom/local roles
!local*/
```

Add `roles_path = roles` to `ansible.cfg` to make sure that roles are searched and downloaded in our local folder.

## Workflow to deploy from the project

1. Clone the project repository
2. Download the external roles: `ansible-galaxy install -r roles/requirements`
3. Launch your playbook: `ansible-playbook -i inventories/dev some_playbook.yml -u ANSIBLE_USER`