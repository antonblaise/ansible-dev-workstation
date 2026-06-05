# ansible-dev-workstation

Ubuntu Developer Workstation Automation with Ansible.

The playbook of this project automates the configuration of a fresh Ubuntu into a developer workstation with these tools:

* Git
* Node.js
* npm
* Python 3
* Docker

Meanwhile, this document records some guidelines on Ansible.

## Ansible playbook hierarchy

```bash
Playbook
 └── Play(s)
      └── Task(s)
           └── Module calls
```

It's not mandatory to name the tasks, but it's the best practice.

## Things we can do before running a playbook

* Compare system state vs desired state.

  ```bash
  ansible-playbook -i inventory.ini setup.yml --ask-become-pass --check
  ```

  For `--check` option, if `become `is `yes `(true), we must use `--ask-become-pass` to prompt us for the super user password. Otherwise, just omit the option.
* Ping the target server/host group of the INI file.

  ```bash
  ansible <server/group> -m ping -i inventory.ini
  ```

  Examples:

  ```
  ansible 127.0.0.1 -m ping --connection=local
  ansible all -m ping -i inventory.ini
  ansible 127.0.0.1 -m ping -i inventory.ini
  ansible servera -m ping -i inventory.ini
  ansible groupa -m ping -i inventory.ini
  ```

## Running a playbook

So now, we remove the `--check` option, and run the command.

```bash
ansible-playbook -i inventory.ini setup.yml 
```

If it's run again when the YAML file is unchanged, then nothing will be changed. This shows that Ansible tasks and modules are **idempotent**.

## Creating Ansible roles

By definition, a `role` in Ansible is a reusable automation structure, such as tasks, variables, handlers, templates, etc.

```bash
ansible-galaxy init <parent folder>/<role name>
```

This command creates a parent folder that contains the folder(s) of the role(s).

* `setup.yml`: orchestration
* `roles/`: building blocks
* `inventory`: target machines

Instead of writing all tasks inside one single `setup.yml`, we instead use the `main.yml` file in `<role name>/tasks` to map the tasks to each role.

Then, in `setup.yml`, we run the tasks assigned to each role by calling the roles. For example:

```yaml
roles:
  - base
  - nodejs
  - docker
  - workspace
```

This runs tasks in `base` role, followed by `nodejs` role, and so on.

In this way, the tasks become reusable. Any other playbook can simply call the role, instead of rewriting the tasks.

## Inventory groups

Ansible inventories are usually grouped, so that they are assigned the appropriate roles, to run the relevant tasks.

Example:

```ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20

[developers]
localhost ansible_connection=local
```

In the playbook, we point to a group using `hosts` keyword, and assign toles to the group. Plus, it's also a common practice to use an individual play for each group instead of running all groups in one single play.

```yaml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes

  roles:
    - nginx
    - docker

- name: Configure Database Servers
  hosts: dbservers
  become: yes

  roles:
    - postgresql

- name: Configure Developer Machines
  hosts: developers
  become: yes

  roles:
    - developer_tools
```

## Ansible ad-hoc command template

We can run ad-hoc Ansible commands for quick admin, troubleshooting and verification.

One ad-hoc Ansible command is equivalent to one task in Ansible playbook.

```bash
ansible <target> -m <module> -a "<module_arguments>" [options]
```

* `<target>`: host, host group, or inventory pattern
* `-m`: module
* `-a`: arguments for the module (optional)
* `[options]`: connection details, inventory, privilege escalation, etc.

**Example 1: Gather system facts**

```bash
ansible all -m setup -i inventory.ini
```

This returns information for all inventories, like:

* ansible_hostname
* ansible_distribution
* ansible_kernel
* ansible_architecture
* ansible_user_id
* ansible_python_version

Add `-a "filter=<search keyword>"` to filter the output results.

**Example 2: Run Linux commands on the target**

```bash
ansible all -m command -a "<command>" -i inventory.ini
```

We can run commands like `df -h` to check disk space, `free -h` to check memory usage, and many more like `systemctl`, `uptime`, etc.
