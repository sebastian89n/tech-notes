`ansible all -a 'hostname -s' -o`

Now using playbook, we register output of hostname command:
```yaml
---
- hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      register: hostname_output
    - name: Show hostname_output
      debug:
        var: hostname_output
    # below debugging only stdout from obj hostname_output
    - name: Show hostname_output.stdout
      debug:
        var: hostname_output.stdout
...
```

E.g. output for `hostname_output.stdout`:
```
TASK [Show hostname_output] ******************************************************************************************************************
ok: [centos1] => {
    "hostname_output.stdout": "centos1"
}
ok: [centos2] => {
    "hostname_output.stdout": "centos2"
}
ok: [centos3] => {
    "hostname_output.stdout": "centos3"
}
ok: [ubuntu1] => {
    "hostname_output.stdout": "ubuntu1"
}
ok: [ubuntu2] => {
    "hostname_output.stdout": "ubuntu2"
}
ok: [ubuntu3] => {
    "hostname_output.stdout": "ubuntu3"
}

```

Using `when`:
```yaml
---
- hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "9"
...
```

Same thing could be changed to do the same on Ubuntu:
`ansible ubuntu1 -m setup -a filter='ansible_distribution*'`
```
ubuntu1 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Debian",
        "ansible_distribution_major_version": "22",
        "ansible_distribution_release": "jammy",
        "ansible_distribution_version": "22.04",
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false
}
```

Example:
```yaml
---
- hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: ( ansible_distribution == "CentOS" and ansible_distribution_major_version >= "8" ) or
            ( ansible_distribution == "Ubuntu" and ansible_distribution_major_version >= "20" )
...
```

Or using list format:
```yaml
---
- hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: 
        - ansible_distribution == "CentOS" 
        - ansible_distribution_major_version | int >= 8
...
```

It is useful to combine `when` and `register`:
```yaml
---
- hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: 
        - ansible_distribution == "CentOS" 
        - ansible_distribution_major_version | int >= 8
      register: command_register

    - name: Show register
      debug:
        var: command_register
...
```

Ubuntu hosts will be skipped in the playbook above.

Output:
```
ok: [ubuntu3] => {
    "command_register": {
        "changed": false,
        "false_condition": "ansible_distribution == \"CentOS\"",
        "skip_reason": "Conditional result was False",
        "skipped": true
    }
}
```

For parsing output best to rely on `changed` field to determine whether something was applied or not.

```yaml
---
- hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: 
        - ansible_distribution == "CentOS" 
        - ansible_distribution_major_version | int >= 8
      register: command_register

    - name: Install patch when changed
      yum:
        name: patch
        state: present
      when: command_register.changed
# or when: command_register is changed (cleaner)
...
```

Above, apply patch only if command_register.changed is true.

Basically it is like registering an output of the module command and assigning it to a fact that can be later displayed or queried upon.

```yaml
---
- hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: 
        - ansible_distribution == "CentOS" 
        - ansible_distribution_major_version | int >= 8
      register: command_register

    - name: Install patch when changed
      yum:
        name: patch
        state: present
      when: command_register is changed

    - name: Install patch when skipped
      apt:
        name: patch
        state: present
      when: command_register is skipped
...
```