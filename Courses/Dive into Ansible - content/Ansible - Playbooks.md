- Ansible Playbooks utilise YAML, as human reaedable, data-serialisation language
- Easy to use, easy to read, great for collaboration
- Reading and writing of YAML, supported in most major programming languages
- Often seen with a .yml or .yaml extension, .yaml is officially the recommended extension since 2006

```yaml
---
-
  hosts: centos
  user: root
  tasks:
    - name: Configure a MOTD (Message of the day)
      copy:
        src: centos_motd
        dest: /etc/motd
...

```

`ansible-playbook motd_playbook.yaml`

Measure time with:
`time ansible-playbook motd_playbook.yaml`

Executes 2 tasks:
- Gathering Facts (hidden/default module run by setup module)
- copy module

Target Options:
- `become`
- `conection`
- `gather_facts`

Copy `content` into files and disable `gather_facts`:x


```yaml
---
-
  hosts: centos
  user: root
  gather_facts: False
  tasks:
    - name: Configure a MOTD (Message of the day)
      copy:
        content: Welcome to CentOS o/
        dest: /etc/motd
...
```

Usage of `jinja2` variables:
```yaml
---
-
  hosts: centos
  user: root
  gather_facts: False
  vars:
    motd: "Welcome to CentOS Linux - Ansible o/\n"
  tasks:
    - name: Configure a MOTD (Message of the day)
      copy:
        content: "{{ motd }}"
        dest: /etc/motd
...
```

Variables can be passed via command line as well:
`ansible-playbook motd_playbook.yaml -e 'motd="Message inline\n"'`

**Handlers are only executed once, at the end of the tasks, when there's a change.**
Using handlers:
```yaml
-
  hosts: centos
  user: root
  gather_facts: False
  vars:
    motd: "Welcome to CentOS Linux - Ansible o/\n"
  tasks:
    - name: Configure a MOTD (Message of the day)
      copy:
        content: "{{ motd }}"
        dest: /etc/motd
      notify: MOTD changed
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
...

```

Chec Facts available:
`ansible all -i centos2, -m setup | more`
`ansible all -i centos2,ubuntu2, -m setup | grep ansible_distribution`

Using info from Facts:
```yaml
---
-
  hosts: linux
  user: root
  gather_facts: True
  vars:
    motd_centos: "Welcome to CentOS Linux - Ansible o/\n"
    motd_ubuntu: "Welcome to Ubuntu Linux - Ansible o/\n"
  tasks:
    - name: Configure a MOTD (Message of the day)
      copy:
        content: "{{ motd_centos }}"
        dest: /etc/motd
      notify: MOTD changed
      when: ansible_distribution == "CentOS"
    - name: Configure a MOTD (Message of the day)
      copy:
        content: "{{ motd_ubuntu }}"
        dest: /etc/motd
      notify: MOTD changed
      when: ansible_distribution == "Ubuntu"
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
...
```

Copy files and preserve mode:
```yaml
---
-
  hosts: ubuntu
  tasks:
    - name: Copy 60-ansible-motd to /etc/update-motd.d
      copy:
        src: 60-ansible-motd
        dest: /etc/update-motd.d/60-ansible-motd
        mode: preserve
      notify: MOTD changed
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
...
```