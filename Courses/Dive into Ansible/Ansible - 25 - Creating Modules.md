>While Ansible provides a rich set of built-in modules for tasks such as file manipulation, package management, and user configuration, there are times when you may need functionality that isn't covered out of the box. In such cases, creating a custom Ansible module allows you to extend Ansible's capabilities to meet your specific needs.
>
>Custom modules are especially useful when:
>- You need to integrate with a proprietary system or internal API.
>- You want more control or flexibility than what existing modules offer.
>-  You're looking to simplify complex tasks into reusable components.
> 
>Custom modules can be written in any language (such as Python, Bash, or even compiled binaries), but Python is the most common and fully supported, thanks to Ansible's module development utilities.
  
**Details:** https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html

Clone ansible:
`git clone https://github.com/ansible/ansible.git`
`ansible --version`
`git checkout v2.17.9`
(make sure checkout ansible version corresponds to the version used)

**Running developed modules via test-module: **

>This script is for testing modules without running through the entire guts of ansible, and is very helpful for developing modules

`vim ~/ansible/hacking/test-module.py` 

`~/ansible/hacking/test-module -m ~/ansible/lib/ansible/modules/command.py -a hostname` -> this did not work locally for me

`python3 ~/ansible/hacking/test-module.py -m ~/ansible/lib/ansible/modules/command.py -a hostname`

**Creating custom modules:**
`icmp.sh`
```bash
#!/bin/bash

ping -c 1 127.0.0.1 >/dev/null 2>/dev/null

if [ $? == 0 ];
  then
  echo "{\"changed\": true, \"rc\": 0}"
else
  echo "{\"failed\": true, \"msg\": \"failed to ping\", \"rc\": 1}"
fi
```

or later version that accepts param:
```bash
#!/bin/bash

# Capture inputs, these are passed as a file to the module
source $1 >/dev/null 2>&1

# Set our variables, set default if not assigned
TARGET=${target:-127.0.0.1}

ping -c 1 ${TARGET} >/dev/null 2>/dev/null

if [ $? == 0 ];
  then
  echo "{\"changed\": true, \"rc\": 0}"
else
  echo "{\"failed\": true, \"msg\": \"failed to ping\", \"rc\": 1}"
fi
```

`python3 ~/ansible/hacking/test-module.py -m icmp.sh -a "target=centos1"`

Custom ansible modules can be written in any languages as long as they return proper json.

---
Ansible will look for custom modules in `library` folder relative to our playbook.

```bash
ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Modules/05$ find .
.
./ansible.cfg
./group_vars
./group_vars/centos
./group_vars/ubuntu
./host_vars
./host_vars/centos1
./host_vars/ubuntu-c
./hosts
./icmp_fail_playbook.yaml
./icmp_playbook.yaml
./library
./library/icmp
```

`icmp_playbook.yaml`
```yaml
---
- hosts: linux
  tasks:

    - name: Test icmp module
      icmp:
        target: 127.0.0.1
...
```

Example of custom module in python:
`icmp.py`
```python
#!/usr/bin/python3

ANSIBLE_METADATA = {
    'metadata_version': '1.1',
    'status': ['preview'],
    'supported_by': 'community'
}

DOCUMENTATION = '''
---
module: icmp

short_description: simple module for icmp ping

version_added: "2.10"

description:
    - "simple module for icmp ping"

options:
    target:
        description:
            - The target to ping
        required: true

author:
    - James Spurin (@spurin)
'''

EXAMPLES = '''
# Ping an IP
- name: Ping an IP
  icmp:
    target: 127.0.0.1

# Ping a host
- name: Ping a host
  icmp:
    target: centos1
'''

RETURN = '''
'''

from ansible.module_utils.basic import AnsibleModule

def run_module():
    # define the available arguments/parameters that a user can pass to
    # the module
    module_args = dict(
        target=dict(type='str', required=True)
    )

    # seed the result dict in the object
    # we primarily care about changed and state
    # change is if this module effectively modified the target
    # state will include any data that you want your module to pass back
    # for consumption, for example, in a subsequent task
    result = dict(
        changed=False
    )

    # the AnsibleModule object will be our abstraction working with Ansible
    # this includes instantiation, a couple of common attr would be the
    # args/params passed to the execution, as well as if the module
    # supports check mode
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    # if the user is working with this module in only check mode we do not
    # want to make any changes to the environment, just return the current
    # state with no modifications
    if module.check_mode:
        return result

    # manipulate or modify the state as needed (this is going to be the
    # part where your module will do what it needs to do)
    ping_result = module.run_command('ping -c 1 {}'.format(module.params['target']))

    # use whatever logic you need to determine whether or not this module
    # made any modifications to your target
    if module.params['target']:
        result['debug'] = ping_result
        result['rc'] = ping_result[0]
        if result['rc']:
          result['failed'] = True
          module.fail_json(msg='failed to ping', **result)
        else:
          result['changed'] = True
          module.exit_json(**result)

def main():
    run_module()

if __name__ == '__main__':
    main()
~
```