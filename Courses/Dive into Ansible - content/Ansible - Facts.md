- The setup module and how this relates to fact gathering
- Filtering for specific facts
- The creation of custom facts
- The execution of custom facts
- How custom facts can be used in environment

`ansible centos1 -m setup -a 'gather_subset=network' | less`

`ansible centos1 -m setup -a 'gather_subset=!all,!min,network' | less`
`!all` and `!min` - exclamation is used to exclude, specific facts sets

**Filtering:**
`ansible centos1 -m setup -a 'filter=ansible_memfree_mb'`
`ansible centos1 -m setup -a 'filter=ansible_mem*'` - using wild card

Facts about our hosts are placed in variable namespace when executing playbook.
E.g.
Output from setup module available whene running playbook with gathering:
```json
centos1 | SUCCESS => {
    "ansible_facts": {
        "ansible_default_ipv4": {
            "address": "172.18.0.7",
            "alias": "eth0",
            "broadcast": "172.18.255.255",
            "gateway": "172.18.0.1",
            "interface": "eth0",
            "macaddress": "aa:15:2d:4c:65:ac",
            "mtu": 1500,
            "netmask": "255.255.0.0",
            "network": "172.18.0.0",
            "prefix": "16",
            "type": "ether"
        },
[...]
```

Retrieving information in the playbook:
```yaml
---

  hosts: all
  tasks:
    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"
...
```

**Ansible idiom(ansible_facts)** - any module returning a dictionary of `ansible_facts` is added to the root of the facts namespace

### Custom facts
- can be written in nay language
- returns a JSON or ini structure
- by default expects to use `/etc/ansible/facts.d` - can be less desired if you are not running as a root and want to keep your configs outside

getdate1.fact (json example)
```bash
#!/bin/bash
echo {\""date\"" : \""`date`\""}
```

getdate2.fact (ini example)
```bash
#!/bin/bash
echo [date]
echo date=`date`
```

Such files can be copied to `/etc/ansible/facts.d` folder.
`sudo cp getdate* /etc/ansible/facts.d/`

`ansible ubuntu-c -m setup -a 'filter=ansible_local'`

custom facts are visible in `ansible_local`:
```json
{
    "ansible_facts": {
        "ansible_local": {
            "getdate1": {
                "date": "Tue May 20 06:41:12 UTC 2025"
            },
            "getdate2": {
                "date": {
                    "date": "Tue May 20 06:41:12 UTC 2025"
                }
            }
        },
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false
}
```

They can be accessed in the playbook:
```yaml
---
-
  hosts: all

  tasks:
    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"

    - name: Show Custom Fact 1
      debug:
        msg: "{{ ansible_local.getdate1.date }}"

    - name: Show Custom Fact 2
      debug:
        msg: "{{ ansible_local.getdate2.date.date }}"
...
```

`ansible-playbook facts_playbook.yaml -l ubunut-c` - l limits the hosts against which we execute the playbook (files with custom facts were copied only on local - ubuntu-c)

**Custom facts are also available in hostvars section:**

```yaml
---
# YAML documents begin with the document separator ---

# The minus in YAML this indicates a list item.  The playbook contains a list
# of plays, with each play being a dictionary
-

  # Hosts: where our play will run and options it will run with
  hosts: all

  # Tasks: the list of tasks that will be executed within the play, this section
  # can also be used for pre and post tasks
  tasks:
    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"

    - name: Show Custom Fact 1
      debug:
        msg: "{{ ansible_local.getdate1.date }}"

    - name: Show Custom Fact 2
      debug:
        msg: "{{ ansible_local.getdate2.date.date }}"

    - name: Show Custom Fact 1 in hostvars
      debug:
        msg: "{{ hostvars[ansible_hostname].ansible_local.getdate1.date }}"

    - name: Show Custom Fact 2 in hostvars
      debug:
        msg: "{{hostvars[ansible_hostname].ansible_local.getdate2.date.date }}"

# Three dots indicate the end of a YAML document
...
```

**Copy custom facts files to other hosts:**
```yaml
---
-
  hosts: linux
  tasks:

    - name: Make Facts Dir
      file:
        path: /etc/ansible/facts.d
        recurse: yes
        state: directory

    - name: Copy Fact 1
      copy:
        src: /etc/ansible/facts.d/getdate1.fact
        dest: /etc/ansible/facts.d/getdate1.fact
        mode: 0755

    - name: Copy Fact 2
      copy:
        src: /etc/ansible/facts.d/getdate2.fact
        dest: /etc/ansible/facts.d/getdate2.fact
        mode: 0755

    - name: Refresh Facts
      setup:

    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"

    - name: Show Custom Fact 1
      debug:
        msg: "{{ ansible_local.getdate1.date }}"

    - name: Show Custom Fact 2
      debug:
        msg: "{{ ansible_local.getdate2.date.date }}"

    - name: Show Custom Fact 1 in hostvars
      debug:
        msg: "{{ hostvars[ansible_hostname].ansible_local.getdate1.date }}"

    - name: Show Custom Fact 2 in hostvars
      debug:
        msg: "{{hostvars[ansible_hostname].ansible_local.getdate2.date.date }}"
...
```

**Using custom path to custom facts**

Remove file created previously:
`ansible linux -m file -a 'path=/etc/ansible/facts.d/getdate1.fact state=absent'`
`ansible linux -m file -a 'path=/etc/ansible/facts.d/getdate2.fact state=absent'`

Create facts.d in home directory of ansible user(to avoid using root):
```yaml
-
  hosts: linux
  tasks:

    - name: Make Facts Dir
      file:
        path: /home/ansible/facts.d
        recurse: yes
        state: directory
        owner: ansible

    - name: Copy Fact 1
      copy:
        src: facts.d/getdate1.fact
        dest: /home/ansible/facts.d/getdate1.fact
        owner: ansible
        mode: 0755

    - name: Copy Fact 2
      copy:
        src: facts.d/getdate2.fact
        dest: /home/ansible/facts.d/getdate2.fact
        owner: ansible
        mode: 0755

    - name: Reload Facts
      setup:
        fact_path: /home/ansible/facts.d

    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"

    - name: Show Custom Fact 1
      debug:
        msg: "{{ ansible_local.getdate1.date }}"

    - name: Show Custom Fact 2
      debug:
        msg: "{{ ansible_local.getdate2.date.date }}"

    - name: Show Custom Fact 1 in hostvars
      debug:
        msg: "{{ hostvars[ansible_hostname].ansible_local.getdate1.date }}"

    - name: Show Custom Fact 2 in hostvars
      debug:
        msg: "{{hostvars[ansible_hostname].ansible_local.getdate2.date.date }}"
...
```

When reloading facts, we can pass `fact_path` that specifies location:
```yaml
    - name: Reload Facts
      setup:
        fact_path: /home/ansible/facts.d
```

Cleanup:
`ansible linux -m file -a 'path=/home/ansible/facts.d state=absent'`