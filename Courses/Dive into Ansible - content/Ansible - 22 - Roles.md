>Ansible **roles** are a structured way to organize playbook content by grouping related tasks, variables, handlers, templates, and files into reusable units. They promote clean, modular automation and make it easier to share and maintain code across projects. Roles can be used in playbooks with the `roles` keyword and are commonly stored under the `roles/` directory. 
>
>You can also manage roles using the `ansible-galaxy` **command-line tool**, which allows you to **install, create, or share roles**, typically from the **Ansible Galaxy repository**, but also from other sources like

- Using Roles
- The Role Structure
- How to create Roles with Ansible Galaxy
- How to move an existing project, to a Role
- Role Execution
- Role Parameters
- Role Dependencies

**Benefits of using Roles:**
- Larger projects are made easier to manage
- Roles are grouped, with logical structure, making them easier to share
- Roles can be written, to specific requirements, for example a web server role, a DNS role or patching role
- Roles can be developed independetly, in parallel by different entities
- Templates, Vars and Files, have designated directories and inclusion is simplified
- Roles can have dependencies on other roles, therefore providing automated inclusion

**Role directory structure:**
```
example-role/
├── README.md
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
├── vars/
│   └── main.yml
```
(top level directory name defines name of the role)

**📁 `tasks`**
Contains the main list of tasks to execute in the role, usually starting from `main.yml`.  
**Put:** Task definitions, role logic, `include_tasks` or `import_tasks`.

**📁 `handlers`**
Stores handlers — tasks triggered by `notify` directives in other tasks.  
**Put:** Service restarts, reloads, or other deferred actions in `main.yml`.

**📁 `defaults`**
Defines default variable values with the **lowest precedence**.  
**Put:** `main.yml` containing safe, overridable defaults for the role.

**📁 `vars`**
Defines variables with **higher precedence** than defaults.  
**Put:** `main.yml` with required or fixed role-specific variables.

**📁 `files`**
Used for copying static files to the target system using the `copy` module.  
**Put:** Static resources like scripts, config files, or binaries.

**📁 `templates`**
Used for placing dynamic configuration files using Jinja2 templating.  
**Put:** Jinja2 templates (e.g., `.j2` files) for rendering with variables.

**📁 `meta`**
Contains metadata about the role such as author, dependencies, and platform support.  
**Put:** `main.yml` with role metadata, especially for Ansible Galaxy.

**📁 `tests`**
Used to test the role manually or in CI.  
**Put:** `test.yml` (test playbook), `inventory` (sample inventory).

**📁 `README.md` (optional but recommended)**
Describes the purpose of the role, variables, usage, and examples.  
**Put:** Human-readable documentation for other users or teammates.

---
**Initialize role structure:**
`ansible-galaxy init nginx`

It creates an empty role sturcture. Old nginx project when moved to role looks like this:
```bash
ansible@ubuntu-c:~/diveintoansible/Structuring Ansible Playbooks/Using Roles/03$ find .
.
./ansible.cfg
./group_vars
./group_vars/centos
./group_vars/ubuntu
./host_vars
./host_vars/centos1
./host_vars/ubuntu-c
./hosts
./nginx
./nginx/.travis.yml
./nginx/README.md
./nginx/defaults
./nginx/defaults/main.yml
./nginx/files
./nginx/files/playbook_stacker.zip
./nginx/handlers
./nginx/handlers/main.yml
./nginx/meta
./nginx/meta/main.yml
./nginx/tasks
./nginx/tasks/main.yml
./nginx/templates
./nginx/templates/index.html-ansible_managed.j2
./nginx/templates/index.html-base.j2
./nginx/templates/index.html-easter_egg.j2
./nginx/templates/index.html-logos.j2
./nginx/templates/index.html.j2
./nginx/tests
./nginx/tests/inventory
./nginx/tests/test.yml
./nginx/vars
./nginx/vars/main.yml
./nginx_playbook.yaml
```

`nginx_playbook.yml` (main file, outside of nginx role folder)
```yaml
---
- hosts: linux

  # Roles: list of roles to be imported into the play 
  roles:
    - nginx
...
```

`nginx` -> `handlers` -> `main.yml`
```yaml
---
# handlers file for nginx
- name: Check HTTP Service
  uri:
    url: http://{{ ansible_default_ipv4.address }}
    status_code: 200
```

`nginx` -> `vars`-> `main.yml`
```yaml
---
# vars file for nginx
centos_logo: 'logo content'
```

`nginx` -> `tasks` -> `main.yaml`
```yaml
---
# tasks file for nginx
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
```
**Note:** because we are using role, there's no need to specify template directory

In `diveintoansible/Structuring Ansible Playbooks/Using Roles/05` role above is split into smaller roles.

In which `nginx_webapp_playbook.yaml` looks afterwards like below:
```yaml
---
- hosts: linux

  # Roles: list of roles to be imported into the play 
  roles:
    - nginx
    - webapp
...
```

or using object format, where we can override the variable using jinja2:
```yaml
---
- hosts: linux
  
  # Roles: list of roles to be imported into the play 
  roles:
    - nginx
    - { role: webapp, target_dir: "{%- if ansible_distribution == 'CentOS' -%}/usr/share/nginx/html{%- elif ansible_distribution == 'Ubuntu' -%}/var/www/html{%- endif %}" }
...
```
(doesn't seem like the greatest approach)

**Dependencies:**
`diveintoansible/Structuring Ansible Playbooks/Using Roles/08/webapp/meta`

in `meta` folder of `webapp` role we can define dependencies:
```yaml
  # If the issue tracker for your role is not on github, uncomment the
  # next line and provide a value
  # issue_tracker_url: http://example.com/issue/tracker

  # Choose a valid license ID from https://spdx.org - some suggested licenses:
  # - BSD-3-Clause (default)
  # - MIT
  # - GPL-2.0-or-later
  # - GPL-3.0-only
  # - Apache-2.0
  # - CC-BY-4.0
  license: license (GPL-2.0-or-later, MIT, etc)

  min_ansible_version: 2.1

  # If this a Container Enabled role, provide the minimum Ansible Container version.
  # min_ansible_container_version:

  #
  # Provide a list of supported platforms, and for each platform a list of versions.
  # If you don't wish to enumerate all versions for a particular platform, use 'all'.
  # To view available platforms and versions (or releases), visit:
  # https://galaxy.ansible.com/api/v1/platforms/
  #
  # platforms:
  # - name: Fedora
  #   versions:
  #   - all
  #   - 25
  # - name: SomePlatform
  #   versions:
  #   - all
  #   - 1.0
  #   - 7
  #   - 99.99

  galaxy_tags: []
    # List tags for your role here, one per line. A tag is a keyword that describes
    # and categorizes the role. Users find roles by searching for tags. Be sure to
    # remove the '[]' above, if you add tags to this list.
    #
    # NOTE: A tag is limited to a single word comprised of alphanumeric characters.
    #       Maximum 20 tags per role.

dependencies:
  - nginx
  # List your role dependencies here, one per line. Be sure to remove the '[]' above,
  # if you add dependencies to this list.
```

If we have `nginx` role dependency specified in `webapp` role then we no longer have to specify it in the `nginx_webapp_playbook.yaml`:
```yaml
---
- hosts: linux
  
  # Roles: list of roles to be imported into the play 
  roles:
    - { role: webapp, target_dir: "{%- if ansible_distribution == 'CentOS' -%}/usr/share/nginx/html{%- elif ansible_distribution == 'Ubuntu' -%}/var/www/html{%- endif %}" }
...
```