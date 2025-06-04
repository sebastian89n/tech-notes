>`ansible.cfg` is the main configuration file for Ansible, used to define settings that control the behavior of Ansible commands and playbooks. It allows you to customize various aspects such as inventory paths, SSH connection options, privilege escalation methods, and output formatting. This file can exist globally (`/etc/ansible/ansible.cfg`), per-user (`~/.ansible.cfg`), or per-project (in the root of your Ansible project directory), with the latter taking highest precedence.

**ansible.cfg - Common and Important Configuration Parameters**
```ini
[defaults]
# Inventory file (can be static or dynamic)
inventory = ./inventory

# Set the default remote user
remote_user = your_user

# Set the private SSH key file
private_key_file = ~/.ssh/id_rsa

# Control output verbosity (0-4)
verbosity = 0

# Enable/disable host key checking
host_key_checking = False

# Set number of parallel forks (concurrent hosts)
forks = 10

# Timeout for SSH connections
timeout = 10

# Whether to gather facts automatically
gathering = smart
gather_subset = all

# Use the pipelining feature to reduce SSH operations
pipelining = True

# Roles path for searching roles
roles_path = ./roles

# Callback plugin to use for output
stdout_callback = yaml  # (can also be json, minimal, etc.)

# Log file for ansible output
log_path = ./ansible.log

# Display skipped hosts
display_skipped_hosts = True

# Show differences when changing files
diff = True

[inventory]
# Enable caching of inventory data
enable_plugins = host_list, script, auto, yaml, ini

[privilege_escalation]
# Enable privilege escalation (e.g. sudo)
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
# Reuse SSH connections for speed
ssh_args = -o ControlMaster=auto -o ControlPersist=60s

# Set a control path for SSH multiplexing
control_path = ~/.ansible/cp/ansible-ssh-%%h-%%p-%%r

# Increase pipelining performance
pipelining = True

# SSH timeout
timeout = 30
```

**Ansible configuration files (in order of priority)**
1. `ANSIBLE_CONFIG` (Environment Variable, with a filename target)
2. `./ansible.cfg` (an ansible.cfg file, in the current directory) -> **recommended**
3. `~/.ansible.cfg` (a hidden file, called .ansible.cfg, in the users home dir) 
4. `/etc/ansible/ansible.cfg` (typically provided, through package or system)

(4)

```bash
root@ubuntu-c:~# mkdir /etc/ansible
root@ubuntu-c:~# touch /etc/ansible/ansible.cfg
```

```bash
root@ubuntu-c:~# ansible --version
ansible [core 2.17.4]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```
(highlights config file)

(3) Switching to use home directory .ansible.cfg:

```bash
ansible@ubuntu-c:~$ cd ~
ansible@ubuntu-c:~$ touch .ansible.cfg
ansible@ubuntu-c:~$ ansible --version
ansible [core 2.17.4]
  config file = /home/ansible/.ansible.cfg
  configured module search path = ['/home/ansible/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /home/ansible/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```

(2)
```bash
ansible@ubuntu-c:~$ mkdir testdir
ansible@ubuntu-c:~$ cd testdir/
ansible@ubuntu-c:~/testdir$ touch ansible.cfg
ansible@ubuntu-c:~/testdir$ ansible --version
ansible [core 2.17.4]
  config file = /home/ansible/testdir/ansible.cfg
  configured module search path = ['/home/ansible/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /home/ansible/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```

(1)
```bash
ansible@ubuntu-c:~/testdir$ touch this_is_my_example_ansible.cfg
ansible@ubuntu-c:~/testdir$ export ANSIBLE_CONFIG=/home/ansible/testdir/this_is_my_example_ansible.cfg 
ansible@ubuntu-c:~/testdir$ ansible --version
ansible [core 2.17.4]
  config file = /home/ansible/testdir/this_is_my_example_ansible.cfg
  configured module search path = ['/home/ansible/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /home/ansible/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
ansible@ubuntu-c:
```

`UNSET ANSIBLE_CONFIG` to remove environment variable