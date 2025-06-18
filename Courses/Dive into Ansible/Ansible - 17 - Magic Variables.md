> Magic (or special) variables in Ansible are predefined variables that provide context about the playbook execution, such as `inventory_hostname`, `hostvars`, or `ansible_play_hosts`. They are automatically available and used to control or reference internal details like hostnames, group membership, and play scope.

https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html

```yaml
---
- hosts: all
  tasks:
    - name: Using template, create a remote file that contains all variables available to the play
      template:
        src: templates/dump_variables
        dest: /tmp/ansible_variables

    - name: Fetch the templated file with all variables, back to the control host
      fetch:
        src: /tmp/ansible_variables
        dest: "captured_variables/{{ ansible_hostname }}"
        flat: yes

    - name: Clean up left over files
      file: 
        name: /tmp/ansible_variables
        state: absent
...
```

`dump_variables` file template:
```
PLAYBOOK VARS (Ansible vars):

{{ vars | to_nice_yaml }}
```

