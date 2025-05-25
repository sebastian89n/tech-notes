Old example:
```yaml
---
- hosts: linux
  vars:
    motd_centos: "Welcome to CentOS Linux - Ansible Rocks\n"
    motd_ubuntu: "Welcome to Ubuntu Linux - Ansible Rocks\n"
  tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        content: "{{ motd_centos }}"
        dest: /etc/motd
      notify: MOTD changed
      when: ansible_distribution == "CentOS"

    - name: Configure a MOTD (message of the day)
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

Improved example using facts:
```yaml
---
- hosts: linux
  tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        content: "Welcome to {{ ansible_distribution }} Linux - Ansible Rocks\n"
        dest: /etc/motd
      notify: MOTD changed
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
...
```

Improved further using loop:
```yaml
---
- hosts: linux
  tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        content: "Welcome to {{ item }} Linux - Ansible Rocks!\n"
        dest: /etc/motd
      notify: MOTD changed
      with_items: [ 'CentOS', 'Ubuntu' ]
      when: ansible_distribution == item
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
...
```

Continue rev 04