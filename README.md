# ansible-dev-workstation

Ubuntu Developer Workstation Automation with Ansible.

This playbook automates the configuration of a fresh Ubuntu into a developer workstation with these tools:

* Git
* Node.js
* npm
* Python 3
* Docker

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
