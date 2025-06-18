> `delegate_to` in Ansible allows a specific task to run on a different host than the one targeted by the play. It's useful for centralizing actions like logging, DNS updates, or interacting with external systems. Only the delegated task runs on the specified host; the rest of the play runs on the original target.

```yaml
---
- hosts: ubuntu-c
  gather_facts: False
  tasks:
    - name: Generate an OpenSSH keypair for ubuntu3
      openssh_keypair:
        path: ~/.ssh/ubuntu3_id_rsa

- hosts: linux
  gather_facts: False

  tasks:
    - name: Copy ubuntu3 OpenSSH keypair with permissions
      copy:
        owner: root
        src: "{{ item.0 }}"
        dest: "{{ item.0 }}"
        mode: "{{ item.1 }}"
      with_together:
        - [ ~/.ssh/ubuntu3_id_rsa, ~/.ssh/ubuntu3_id_rsa.pub ]
        - [ "0600", "0644" ]

- hosts: ubuntu3
  gather_facts: False

  tasks:
    - name: Add public key to the ubuntu3 authorized_keys file
      authorized_key:
        user: root
        state: present
        key: "{{ lookup('file', '~/.ssh/ubuntu3_id_rsa.pub') }}"

- hosts: all
  gather_facts: False
  tasks:
    - name: Check that ssh can connect to ubuntu3 using the ssh tool
      command: ssh -i ~/.ssh/ubuntu3_id_rsa -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@ubuntu3 date
      changed_when: False
      ignore_errors: True

- hosts: ubuntu-c, centos1, ubuntu1
  # Serial is important as we are writing to a single file
  serial: 1
  tasks:
    - name: Add host to /etc/hosts.allow for sshd
      lineinfile:
        path: /etc/hosts.allow
        line: "sshd: {{ ansible_hostname }}.diveinto.io"
        create: True
      delegate_to: ubuntu3

- hosts: all
  gather_facts: False

  tasks:
    - name: Check that ssh can connect to ubuntu3 using the ssh tool
      command: ssh -i ~/.ssh/ubuntu3_id_rsa -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@ubuntu3 date
      changed_when: False
      ignore_errors: True

- hosts: ubuntu3
  gather_facts: False
  tasks:
    - name: Drop SSH connectivity from everywhere else
      lineinfile:
        path: /etc/hosts.deny
        line: "sshd: ALL"
        create: True

- hosts: all
  gather_facts: False
  tasks:
    - name: Check that ssh can connect to ubuntu3 using the ssh tool
      command: ssh -i ~/.ssh/ubuntu3_id_rsa -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@ubuntu3 date
      changed_when: False
      ignore_errors: True

- hosts: ubuntu-c, centos1, ubuntu1
  # Serial is important as we are writing to a single file
  serial: 1
  tasks:
    - name: Remove specific host entries in /etc/hosts.allow for sshd
      lineinfile:
        path: /etc/hosts.allow
        line: "sshd: {{ ansible_hostname }}.diveinto.io"
        state: absent
      delegate_to: ubuntu3

- hosts: ubuntu3
  gather_facts: False
  tasks:
    - name: Allow SSH connectivity from everywhere
      lineinfile:
        path: /etc/hosts.deny
        line: "sshd: ALL"
        state: absent
...
```

**Steps from the playbook:**
- create a new dedicated ssh key pair to be used by ubuntu3 only
- copy key pair to all our target system using linux group
- install public key using `authorized_key` on ubuntu3
- gathering facts on ubuntu-c, centos1 and ubuntu1 and through delegation, add a line to `/etc/hosts.allow` on `ubuntu3` with those values
- adding a line in `hosts.deny` to deny access anyone who is not in the `hosts.allow`
- clean-up