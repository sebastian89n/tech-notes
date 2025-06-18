> Blocks in Ansible group multiple tasks together, allowing shared control structures like `when`, `become`, `rescue`, and `always`. They're useful for applying conditional logic or error handling to a set of tasks as a unit.
> 
> In Ansible, `when`, `become`, `rescue`, and `always` act like control flow tools, similar to `if`, `try`, `catch`, and `finally` in traditional programming languages.

- How to group multiple tasks, into a single block
- Rescue
- Always

**Basic example:**
```yaml
---
- hosts: linux
  tasks:
    - name: A block of modules being executed
      block:
        - name: Example 1
          debug:
            msg: Example 1

        - name: Example 2
          debug:
            msg: Example 2

        - name: Example 3
          debug:
            msg: Example 3
...
```

**Example with when, with:**
```yaml
---
- hosts: linux
  tasks:
    - name: A block of modules being executed
      block:
        - name: Example 1 CentOS only
          debug:
            msg: Example 1 CentOS only
          when: ansible_distribution == 'CentOS'

        - name: Example 2 Ubuntu only
          debug:
            msg: Example 2 Ubuntu only
          when: ansible_distribution == 'Ubuntu'

        - name: Example 3 with items
          debug:
            msg: "Example 3 with items - {{ item }}"
          with_items: ['x', 'y', 'z'] 
...
```

```yaml
---
- hosts: linux
  tasks:

    - name: Install patch and python3-dnspython
      block:
        - name: Install patch
          package:
            name: patch

        - name: Install python3-dnspython
          package:
            name: python3-dnspython

      rescue:
        - name: Rollback patch
          package:
            name: patch
            state: absent

        - name: Rollback python3-dnspython
          package:
            name: python3-dnspython
            state: absent

      always:
        - debug:
            msg: This always runs, regardless
...
```