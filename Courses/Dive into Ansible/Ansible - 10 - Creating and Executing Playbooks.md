**ansible.cfg:**
```ini
[defaults]
inventory = hosts
host_key_checking = False
ansible_managed = Managed by Ansible - file:{file} - host:{host} - uid:{uid}
```

**Using dnf/apt/yum modules:**
```yaml
---
- hosts: linux
  vars:
    update_cache: yes
    state: latest
  tasks:
    - name: Install EPEL
      yum:
        name: epel-release
        state: latest
        update_cache: yes
      when: ansible_distribution == 'CentOS'
    - name: Install Nginx CentOS
      dnf:
        name: nginx
        state: latest
        update_cache: yes
      when: ansible_distribution == 'CentOS'
    - name: Install Nginx Ubuntu
      apt:
        name: nginx
        state: latest
        update_cache: yes
      when: ansible_distribution == 'Ubuntu'
...
```

**Here example of**:
- using package module
- using service module to restart nginx
- notifying handler to use uri module to check if nginx has restarted
- `nginx_root_location` is specified in group_vars because it's different between CentOS/Ubuntu
	- centos: `/usr/share/nginx/html`
	- ubuntu: `/var/www/html`
- use template module to template .j2 files into `index.html` on nginx
- install unzip

```yaml
---
- hosts: linux
  vars_files:
    - vars/logos.yaml
  tasks:
    - name: Install EPEL
      yum:
        name: epel-release
        state: latest
        update_cache: yes
      when: ansible_distribution == 'CentOS'
    - name: Install nginx
      package:
        name:
          - nginx
        state: latest
    - name: Restart nginx
      service: 
        name: nginx
        state: restarted
      notify: Check HTTP Service
    - name: Template index.html-base.j2 to index.html on target
      template:
        src: index.html-base.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
    - name: Override template index.html-ansible_managed.j2 to index.html on target
      template:
        src: index.html-ansible_managed.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
    - name: Override template index.html-logos.j2 to index.html on target
      template:
        src: index.html-logos.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
    - name: Override template index.html-easter_egg.j2 to index.html on target
      template:
        src: index.html-easter_egg.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
    - name: Install unzip
      package:
        name: unzip
        state: latest
    - name: Unarchive playbook stacker game
      unarchive:
        src: playbook_stacker.zip
        dest: "{{ nginx_root_location }}"
        mode: 0755
  handlers:
    - name: Check HTTP Service
      uri:
        url: http://{{ ansible_default_ipv4.address }}
        status_code: 200
...
```