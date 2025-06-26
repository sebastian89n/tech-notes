>In Ansible, **tags** are used to label tasks, roles, or plays so you can selectively run parts of your playbook using the `--tags` or `--skip-tags` options. This allows for more efficient execution by running only the specific tasks you need, such as "install", "configure", or "restart". Tags help streamline development, debugging, and partial deployments.

- Using Tags 
- Segmentation with Tags
- Execution with Tags
- Skipping with Tags
- Playbook Tags
- Special Tags
- Tag Inheritance

```yaml
---
- hosts: linux
  vars_files:
    - vars/logos.yaml

  tasks:
    - name: Install EPEL
      yum:
        name: epel-release
        update_cache: yes
        state: latest
      when: ansible_distribution == 'CentOS'
      tags:
        - install-epel

    - name: Install Nginx
      package:
        name: nginx
        state: latest
      tags:
        - install-nginx

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      notify: Check HTTP Service
      tags:
        - restart-nginx

    - name: Template index.html-easter_egg.j2 to index.html on target
      template:
        src: index.html-easter_egg.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
      tags:
        - deploy-app

    - name: Install unzip
      package:
        name: unzip
        state: latest

    - name: Unarchive playbook stacker game
      unarchive:
        src: playbook_stacker.zip
        dest: "{{ nginx_root_location }}"
        mode: 0755
      tags:
        - deploy-app

  handlers:
    - name: Check HTTP Service
      uri:
        url: http://{{ ansible_default_ipv4.address }}
        status_code: 200 
...
```

Now if we want to run only piece of the full playbook:
`ansible-playbook nginx_playbook.yaml --tags 'install-epel'`

or to execute more tags:
`ansible-playbook nginx_playbook.yaml --tags 'install-epel,restart-nginx'`

`ansible-playbook nginx_playbook.yaml --tags 'deploy-app'`

Run everything except for `deploy-app`:
`ansible-playbook nginx_playbook.yaml --skip-tags 'deploy-app'`

Or to run all:
`ansible-playbook nginx_playbook.yaml --tags 'all'`

**To apply tag on entire playbook:**
```yaml
---
- hosts: linux
  tags:
    - webapp
  
  vars_files:
    - vars/logos.yaml

  tasks:
    - name: Install EPEL
      yum:
        name: epel-release
        update_cache: yes
        state: latest
      when: ansible_distribution == 'CentOS'
      tags:
        - install-epel

    - name: Install Nginx
      package:
        name: nginx
        state: latest
      tags:
        - install-nginx

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      notify: Check HTTP Service
      tags:
        - restart-nginx

    - name: Template index.html-easter_egg.j2 to index.html on target
      template:
        src: index.html-easter_egg.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
      tags:
        - deploy-app

    - name: Install unzip
      package:
        name: unzip
        state: latest

    - name: Unarchive playbook stacker game
      unarchive:
        src: playbook_stacker.zip
        dest: "{{ nginx_root_location }}"
        mode: 0755
      tags:
        - deploy-app

  handlers:
    - name: Check HTTP Service
      uri:
        url: http://{{ ansible_default_ipv4.address }}
        status_code: 200 
...
```

`ansible-playbook nginx_playbook.yaml --tags 'webapp'`

>When you add a **tag to an entire play** (not just individual tasks), **fact gathering becomes bound to that tag**. This means that:
>
>If you run a command like `ansible-playbook site.yml --tags some_task`, and the **play itself has a different tag** (e.g., `play_tag`), then the play — including **fact gathering** — is **skipped entirely** unless `some_task` matches the play’s tag.
>
>So even if a task inside the play matches the tag you passed, **the whole play may be skipped** if the play-level tag isn’t included — and therefore **facts won’t be gathered**.

**It's possible to mitigate it by doing an empty task with hosts:**
```yaml

---
- hosts: linux

- hosts: linux
  tags:
    - webapp
  
  vars_files:
    - vars/logos.yaml

  tasks:
    - name: Install EPEL
      yum:
        name: epel-release
        update_cache: yes
        state: latest
      when: ansible_distribution == 'CentOS'
      tags:
        - install-epel

    - name: Install Nginx
      package:
        name: nginx
        state: latest
      tags:
        - install-nginx

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      notify: Check HTTP Service
      tags:
        - restart-nginx

    - name: Template index.html-easter_egg.j2 to index.html on target
      template:
        src: index.html-easter_egg.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
      tags:
        - deploy-app

    - name: Install unzip
      package:
        name: unzip
        state: latest

    - name: Unarchive playbook stacker game
      unarchive:
        src: playbook_stacker.zip
        dest: "{{ nginx_root_location }}"
        mode: 0755
      tags:
        - deploy-app

  handlers:
    - name: Check HTTP Service
      uri:
        url: http://{{ ansible_default_ipv4.address }}
        status_code: 200 
...
```

**Using `always`:**
```yaml
---
- hosts: linux

- hosts: linux
  tags:
    - webapp
  
  vars_files:
    - vars/logos.yaml

  tasks:
    - name: Install EPEL
      yum:
        name: epel-release
        update_cache: yes
        state: latest
      when: ansible_distribution == 'CentOS'
      tags:
        - install-epel

    - name: Install Nginx
      package:
        name: nginx
        state: latest
      tags:
        - install-nginx

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      notify: Check HTTP Service
      tags:
        - always

    - name: Template index.html-easter_egg.j2 to index.html on target
      template:
        src: index.html-easter_egg.j2
        dest: "{{ nginx_root_location }}/index.html"
        mode: 0644
      tags:
        - deploy-app

    - name: Install unzip
      package:
        name: unzip
        state: latest

    - name: Unarchive playbook stacker game
      unarchive:
        src: playbook_stacker.zip
        dest: "{{ nginx_root_location }}"
        mode: 0755
      tags:
        - deploy-app

  handlers:
    - name: Check HTTP Service
      uri:
        url: http://{{ ansible_default_ipv4.address }}
        status_code: 200 
...
```

`ansible-playbook nginx_playbook.yaml --tags 'install-nginx'`

Above `restart` tag of nginx will be executed as well because it is marked with `always`.

We can skip always with `--skip-tags`:
`ansible-playbook nginx_playbook.yaml --tags 'install-nginx' --skip-tags 'always'`

`ansible-playbook nginx_playbook.yaml -tags 'tagged'` - execute only tagged tasks
`ansible-playbook nginx_playbook.yaml -tags 'untagged'` - execute only untagged tasks
`ansible-playbook nginx_playbook.yaml -tags 'all'` - execute everything

**Tags can be applied against includes and imports:**

```yaml
---
- hosts: ubuntu3
  
- tasks:

    - include_tasks: include_tasks.yaml
      tags:
        - include_tasks

    - import_tasks: import_tasks.yaml
      tags:
        - import_tasks

- import_playbook: import_playbook.yaml
  tags:
    - import_playbook
...
```