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
      with_items: [ 'CentOS', 'Ubuntu' ] # or - notation for lists
      when: ansible_distribution == item
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
...
```

Looping is very useful for tasks like creating users:
```yaml
---
- hosts: linux
  tasks:
    - name: Creating user
      user:
        name: "{{ item }}"
        # state: absent
      with_items: 
        - james
        - hayley
        - lily
        - anwen
...
```

Looping with dict:
```yaml
---
- hosts: linux
  tasks:
    - name: Creating user
      user:
        name: "{{ item.key }}"
        comment: "{{ item.value.full_name }}"
      with_dict: 
        james: 
          full_name: James Spurin
        hayley: 
          full_name: Hayley Spurin
        lily: 
          full_name: Lily Spurin
        anwen:
          full_name: Anwen Spurin
...
```

Usage of `with_subelements` - accepts lists of dictionaries
```yaml
---
- hosts: linux
  tasks:
    - name: Creating user
      user:
        name: "{{ item.1 }}"
        comment: "{{ item.1 | title }} {{ item.0.surname }}"
        # https://docs.ansible.com/ansible/latest/plugins/lookup/password.html
        password: "{{ lookup('password', '/dev/null length=15 chars=ascii_letters,digits,hexdigits,punctuation') | password_hash('sha512') }}"
      with_subelements: 
        - family:
            surname: Spurin
            members:
             - james
             - hayley
             - lily
             - anwen
        - members
...
```

Jinja2 filter `item.1 | title` changes james to James

Usage of `with_nested`("for each of these, do one each of these"):
```yaml
---
- hosts: linux
  tasks:
    - name: Creating user directories
      file:
        dest: "/home/{{ item.0 }}/{{ item.1 }}"
        owner: "{{ item.0 }}"
        group: "{{ item.0 }}"
        state: directory
      with_nested:
        - [ james, hayley, freya, lily, anwen, ana, abhishek, sara ]
        - [ photos, movies, documents ]
...
```

Usage of `with_file` (file content in loops):
```yaml
---
- hosts: linux
  tasks:
    - name: Create authorized key
      authorized_key:
        user: james
        key: "{{ item }}"
      with_file:
        - /home/ansible/.ssh/id_rsa.pub
        - custom_key.pub
...
```

Above snippet adds public key and custom_key to user authorized keys

Usage of `with_sequence`:
```yaml
---
- hosts: linux
  tasks:
    - name: Create sequence directories
      file:
        dest: "{{ item }}"
        state: directory
      with_sequence: start=0 end=100 stride=10 format=/home/james/sequence_%d
...
```

`with_sequence` has to be used as key-value pair.

%d->%x 
```yaml
---
- hosts: linux
  tasks:
    - name: Create hex sequence directories
      file:
        dest: "{{ item }}"
        state: directory
      with_sequence: count=5 format=/home/james/count_sequence_%x
...
```

Usage of `with_random_choice`:
```yaml
---
- hosts: linux
  tasks:
    - name: Create random directory
      file:
        dest: "/home/james/{{ item }}"
        state: directory
      with_random_choice:
        - "google"
        - "facebook"
        - "microsoft"
        - "apple"
...
```

Usage of `until`:
```yaml
---
- hosts: linux
  tasks:
    - name: Run a script until we hit 10
      script: random.sh
      register: result
      retries: 100
      until: result.stdout.find("10") != -1
      # n.b. the default delay is 5 seconds
      delay: 1
...
```
(it can be very useful with the right usage)