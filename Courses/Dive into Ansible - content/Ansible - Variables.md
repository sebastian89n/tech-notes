(numbers corresponds to example in section 4.14 variables module)

**1. Key-value pair**
```yaml
---
  # Hosts: where our play will run and options it will run with
  hosts: centos1
  gather_facts: False
 
  # Vars: variables that will apply to the play, on all target systems
  vars:
    example_key: example value
 
  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test dictionary key value
      debug:
        msg: "{{ example_key }}"
...
```

**2. Dictionary**
```yaml
---
  # Hosts: where our play will run and options it will run with
  hosts: centos1
  gather_facts: False
 
  # Vars: variables that will apply to the play, on all target systems
  vars:
    dict:
      dict_key: This is a dictionary value
 
  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test named dictionary dictionary
      debug:
        msg: "{{ dict }}"
 
    - name: Test named dictionary dictionary key value with dictionary dot notation
      debug:
        msg: "{{ dict.dict_key }}"
 
    - name: Test named dictionary dictionary key value with python brackets notation
      debug:
        msg: "{{ dict['dict_key'] }}"
...
```

**3. Inline dictionary**
```yaml
---
  # Hosts: where our play will run and options it will run with
  hosts: centos1
  gather_facts: False
 
  # Vars: variables that will apply to the play, on all target systems
  vars:
    inline_dict:
      {inline_dict_key: This is an inline dictionary value}
 
  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test named inline dictionary dictionary
      debug:
        msg: "{{ inline_dict }}"
 
    - name: Test named inline dictionary dictionary key value with dictionary dot notation
      debug:
        msg: "{{ inline_dict.inline_dict_key }}"
 
    - name: Test named inline dictionary dictionary key value with brackets notation
      debug:
        msg: "{{ inline_dict['inline_dict_key'] }}"
 
...
```

**4. Normal list**
```yaml
---
  # Hosts: where our play will run and options it will run with
  hosts: centos1
  gather_facts: False
 
  # Vars: variables that will apply to the play, on all target systems
  vars:
    named_list:
      - item1
      - item2
      - item3
      - item4
 
  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test named list
      debug:
        msg: "{{ named_list }}"
 
    - name: Test named list first item dot notation
      debug:
        msg: "{{ named_list.0 }}"
 
    - name: Test named list first item brackets notation
      debug:
        msg: "{{ named_list[0] }}"
...
```

**5. Inline list**
```yaml
---
  # Hosts: where our play will run and options it will run with
  hosts: centos1
  gather_facts: False
 
  # Vars: variables that will apply to the play, on all target systems
  vars:
    inline_named_list:
      [ item1, item2, item3, item4 ]
 
  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test inline named list
      debug:
        msg: "{{ inline_named_list }}"
 
    - name: Test inline named list first item dot notation
      debug:
        msg: "{{ inline_named_list.0 }}"
 
    - name: Test inline named list first item brackets notation
      debug:
        msg: "{{ inline_named_list[0] }}"
 
...
```

**6. External var files using `vars_files`**
```yaml
---
  hosts: centos1
  gather_facts: False
 
  # Vars: variables that will apply to the play, on all target systems
  vars_files:
    - external_vars.yaml
 
  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test external dictionary key value
      debug:
        msg: "{{ external_example_key }}"

    - name: Test external named dictionary dictionary
      debug:
        msg: "{{ external_dict }}"

    - name: Test external named dictionary dictionary key value with dictionary dot notation
      debug:
        msg: "{{ external_dict.dict_key }}"

    - name: Test external named dictionary dictionary key value with brackets notation
      debug:
        msg: "{{ external_dict['dict_key'] }}"
 
    - name: Test external named inline dictionary dictionary
      debug:
        msg: "{{ external_inline_dict }}"
 
    - name: Test external named inline dictionary dictionary key value with dictionary dot notation
      debug:
        msg: "{{ external_inline_dict.inline_dict_key }}"
 
    - name: Test external named inline dictionary dictionary key value with brackets notation
      debug:
        msg: "{{ external_inline_dict['inline_dict_key'] }}"
 
    - name: Test external named list
      debug:
        msg: "{{ external_named_list }}"
 
    - name: Test external named list first item dot notation
      debug:
        msg: "{{ external_named_list.0 }}"
 
    - name: Test external named list first item brackets notation
      debug:
        msg: "{{ external_named_list[0] }}"
 
    - name: Test external inline named list
      debug:
        msg: "{{ external_inline_named_list }}"
 
    - name: Test external inline named list first item dot notation
      debug:
        msg: "{{ external_inline_named_list.0 }}"
 
    - name: Test external inline named list first item brackets notation
      debug:
        msg: "{{ external_inline_named_list[0] }}"
...
```
External file:
```yaml
---
external_example_key: example value

external_dict:
   dict_key: This is a dictionary value

external_inline_dict: 
   {inline_dict_key: This is an inline dictionary value}

external_named_list:
   - item1
   - item2
   - item3
   - item4

external_inline_named_list:
   [ item1, item2, item3, item4 ]
...
```


**7 & 8. Var prompts - prompts users for the variable**
```yaml
  # Hosts: where our play will run and options it will run with
  hosts: centos1
  gather_facts: False
 
  # Vars: variables that will apply to the play, on all target systems
  vars_prompt:
    - name: username
      private: False
 
  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test vars_prompt
      debug:
        msg: "{{ username }}"
 
# Three dots indicate the end of a YAML document
...
```

`private` defines whether it's sensitive var or not (like password)

**9 & 10. Using `hostvars` to access host specific data** (requires field to be present in `hostvars`. `Groupvars`(variables assigned on a group) can be also accessed via `hostvars`)
```yaml
---
  # Hosts: where our play will run and options it will run with
  hosts: centos1
  gather_facts: True
 
  # Vars: variables that will apply to the play, on all target systems

  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test hostvars with an ansible fact and collect ansible_port, dot notation
      debug:
        msg: "{{ hostvars[ansible_hostname].ansible_port }}"

    - name: Test hostvars with an ansible fact and collect ansible_port, dict notation
      debug:
        msg: "{{ hostvars[ansible_hostname]['ansible_port'] }}"
 
# Three dots indicate the end of a YAML document
...
```

**11.  Default value using jinja2 filter**
```yaml
---
  # Hosts: where our play will run and options it will run with
  hosts: centos
  gather_facts: True
 
  # Vars: variables that will apply to the play, on all target systems

  # Tasks: the list of tasks that will be executed within the playbook
  tasks:
    - name: Test hostvars with an ansible fact and collect ansible_port, dot notation, default if not found
      debug:
        msg: "{{ hostvars[ansible_hostname].ansible_port | default('22') }}"

    - name: Test hostvars with an ansible fact and collect ansible_port, dict notation, default if not found
      debug:
        msg: "{{ hostvars[ansible_hostname]['ansible_port'] | default('22') }}"
 
# Three dots indicate the end of a YAML document
...
```

**15. You can use `group_vars` and `host_vars` folders to define variables per group/host**

**17. Extra vars**

**ini format:** `ansible-playbook variables_playbook.yml -e extra_vars_key="extra varas value"`
**json format:** `ansible-playbook variables_playbook.yml -e {"extra_var_key": "extra vars value"}`
**yaml format:** `ansible-playbook variables_playbook.yml -e {extra_vars_key: extra vars value}`

`ansible-playbook variables_playbook.yml -e @extra_vars_file.yaml`
`ansible-playbook variables_playbook.yml -e @extra_vars_file.josn`

**Directories for Hostvars and Groupvars**
- it also possible to use directories for host and group variables in combination with a Yaml file
- Hostvars use directory structure `host_vars/hostname` e.g. `host_vars/ubuntu-c`
- Groupvar use the directory structure `group_vars/group` e.g. `group_vars/ubuntu`
