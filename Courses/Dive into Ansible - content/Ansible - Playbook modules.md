Common playbook modules to remember:
- set_fact
- pause
- prompt
- wait_for
- assemble
- add_host
- group_by
- fetch

1. **Set Fact** - used for gathering facts when executing playbooks. Dynamically add or change facts during execution
```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Set a fact
      set_fact:
        our_fact: Ansible Rocks!

    - name: Show custom fact
      debug:
        msg: "{{ our_fact }}"
...
```

Override ansible-distribution using filter to make it all uppercase:
```yaml
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Set a fact
      set_fact:
        our_fact: Ansible Rocks!
        ansible_distribution: "{{ ansible_distribution | upper }}"

    - name: Show our_fact
      debug:
        msg: "{{ our_fact }}"

    - name: Show ansible_distribution
      debug:
        msg: "{{ ansible_distribution }}"
...
```

Combining setting facts with `when`(instead of using inventory and group_vars):
```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Set our installation variables for CentOS
      set_fact:
        webserver_application_port: 80
        webserver_application_path: /usr/share/nginx/html
        webserver_application_user: root
      when: ansible_distribution == 'CentOS'

    - name: Set our installation variables for Ubuntu
      set_fact:
        webserver_application_port: 8080
        webserver_application_path: /var/www/html
        webserver_application_user: nginx
      when: ansible_distribution == 'Ubuntu'

    - name: Show pre-set distribution based facts
      debug:
        msg: "webserver_application_port:{{ webserver_application_port }} webserver_application_path:{{ webserver_application_path }} webserver_application_user:{{ webserver_application_user }}"
...
```

2. **Pause** - a playbook for a set amount of time or until a prompt is acknowledged

```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Pause our playbook for 10 seconds
      pause:
        seconds: 10
...
```

It can be used as a user prompt:
```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Prompt user to verify before continue
      pause:
        prompt: Please check that the webserver is running, press enter to continue
...
```

Waiting for port 80 to be in use (example 06):
```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Wait for the webserver to be running on port 80
      wait_for:
        port: 80
...
```

`ansible centos3 -m service -a "name=nginx state=stopped"`
`ansible-playbook wait_for_playbook.yaml &` - run in the detached process
`ansible centos3 -m service -a "name=nginx state=started"` -> playbook continues when server starts

3. **Assemble** - allows configuration files to be broken into segments and concatenated to form a destination file. Great to use, when an application or tool, requirees it's configuration as a single file, but you wish to manage it as separate entities

```yaml
---
-
  hosts: ubuntu-c
  tasks:
    - name: Assemble conf.d to sshd_config
      assemble:
        src: conf.d
        dest: sshd_config
...
```

sshd_config:
```ini
## Custom for centos1
Host centos1
  User root
  Port 2222

## Defaults

Port 22
Protocol 2
ForwardX11 yes
GSSAPIAuthentication no
```

To test after running:
`ssh -F sshd_config centos1`

4. **Add Host**
- create new invenetory groups and targets, on the fly
- great, for when a resource is created during execution and you wish to include it in your playbook exection

We start with ubuntu-c, but we create new groups where we add centos1. Then we target first group in a second play:
```yaml
---
-
  hosts: ubuntu-c
  tasks:
    - name: Add centos1 to adhoc_group
      add_host:
        name: centos1
        groups: adhoc_group1, adhoc_group2

-
  hosts: adhoc_group1
  tasks:
    - name: Ping all in adhoc_group1
      ping:
...
```

Another way to format yaml:
```yaml
---
- hosts: ubuntu-c
  tasks:
    - name: Add centos1 to adhoc_group
      add_host:
        name: centos1
        groups: adhoc_group1, adhoc_group2

- hosts: adhoc_group1
  tasks:
    - name: Ping all in adhoc_group1
      ping:
...
```

5. **Group By** - utilise facts to dynamically create, associated groups

Create a custom group on the fly with `custom_` prefix
```yaml
---
- hosts: all
  tasks:
    - name: Create group based on ansible_distribution
      group_by:
        key: "custom_{{ ansible_distribution | lower }}"
- hosts: custom_centos
  tasks:
    - name: Ping all in custom_centos
      ping:

...
```

6. **Fetch** - capture files from remote hosts and targets
```yaml
---
- hosts: centos
  tasks:
    - name: Fetch /etc/redhat-release
      fetch:
        src: /etc/redhat-release
        dest: /tmp/redhat-release
...
```