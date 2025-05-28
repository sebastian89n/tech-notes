>By default, Ansible runs tasks synchronously, holding the connection to the remote node open until the action is completed. This means that, within a playbook, each task blocks the next task by default, and subsequent tasks will not run until the current task is completed. This behavior can create challenges. For example, a task may take longer to complete than the SSH session allows for, causing a timeout. Or you may want a long-running process to execute in the background while you perform other tasks concurrently. Asynchronous mode lets you control how long-running tasks are executed.

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_async.html

With linear strategy:
- Each task runs **in order**, and Ansible waits for **all hosts** to complete **task N** before moving on to **task N+1**.

With free strategy:
- Each host **executes tasks as quickly as possible**, **independently** of other hosts.
- It **does not wait** for all hosts to finish the current task before moving to the next one.


Slow playbook:
```yaml
---
- hosts: linux
  tasks:
    - name: Task 1
      command: /bin/sleep 5

    - name: Task 2
      command: /bin/sleep 5

    - name: Task 3
      command: /bin/sleep 5

    - name: Task 4
      command: /bin/sleep 5

    - name: Task 5
      command: /bin/sleep 5

    - name: Task 6
      command: /bin/sleep 5
...
```

`time ansible-playbook slow_playbook.yaml`

>1 min 16 sec

Only run task for specific host:
```yaml
---
- hosts: linux
  tasks:
    - name: Task 1
      command: /bin/sleep 5
      when: ansible_hostname == 'centos1'

    - name: Task 2
      command: /bin/sleep 5
      when: ansible_hostname == 'centos2'

    - name: Task 3
      command: /bin/sleep 5
      when: ansible_hostname == 'centos3'

    - name: Task 4
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu1'

    - name: Task 5
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu2'

    - name: Task 6
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu3'
...
```

>41 sec

Using async, with value (wait at least 10 sec and poll every 1 sec)

```yaml
---
- hosts: linux
  tasks:
    - name: Task 1
      command: /bin/sleep 5
      when: ansible_hostname == 'centos1'
      async: 10
      poll: 0

    - name: Task 2
      command: /bin/sleep 5
      when: ansible_hostname == 'centos2'
      async: 10
      poll: 0

    - name: Task 3
      command: /bin/sleep 5
      when: ansible_hostname == 'centos3'
      async: 10
      poll: 0

    - name: Task 4
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu1'
      async: 10
      poll: 0

    - name: Task 5
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu2'
      async: 10
      poll: 1

    - name: Task 6
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu3'
      async: 10
      poll: 0
...
```

> 12 sec

but the tasks are still running in the background after returning (can be checked with `ps -ef | grep ssh`)

**Register and debug:**
```yaml
---
- hosts: linux
  tasks:
    - name: Task 1
      command: /bin/sleep 5
      when: ansible_hostname == 'centos1'
      async: 10
      poll: 0
      register: result1

    - name: Task 2
      command: /bin/sleep 5
      when: ansible_hostname == 'centos2'
      async: 10
      poll: 0
      register: result2

    - name: Task 3
      command: /bin/sleep 5
      when: ansible_hostname == 'centos3'
      async: 10
      poll: 0
      register: result3

    - name: Task 4
      command: /bin/sleep 30
      when: ansible_hostname == 'ubuntu1'
      async: 60
      poll: 0
      register: result4

    - name: Task 5
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu2'
      async: 10
      poll: 0
      register: result5

    - name: Task 6
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu3'
      async: 10
      poll: 0
      register: result6

    - name: Show registered context
      debug:
        var: result1

    - name: Show registered context as jinja2
      debug:
        msg: "{{ result1 }}"
...
```

**Register job ids:**
```yaml
---
- hosts: linux
  vars:
    jobids: []
  tasks:
    - name: Task 1
      command: /bin/sleep 5
      when: ansible_hostname == 'centos1'
      async: 10
      poll: 0
      register: result1

    - name: Task 2
      command: /bin/sleep 5
      when: ansible_hostname == 'centos2'
      async: 10
      poll: 0
      register: result2

    - name: Task 3
      command: /bin/sleep 5
      when: ansible_hostname == 'centos3'
      async: 10
      poll: 0
      register: result3

    - name: Task 4
      command: /bin/sleep 30
      when: ansible_hostname == 'ubuntu1'
      async: 60
      poll: 0
      register: result4

    - name: Task 5
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu2'
      async: 10
      poll: 0
      register: result5

    - name: Task 6
      command: /bin/sleep 5
      when: ansible_hostname == 'ubuntu3'
      async: 10
      poll: 0
      register: result6

    - name: Capture Job IDs
      set_fact:
        jobids: >
                {% if item.ansible_job_id is defined -%}
                  {{ jobids + [item.ansible_job_id] }}
                {% else -%}
                  {{ jobids }}
                {% endif %}
      with_items: "{{ [ result1, result2, result3, result4, result5, result6 ] }}"

    - name: Show Job IDs
      debug:
        var: jobids
...
```

**Waits for jobs to finish for specific job ids, retries up to 30 times:**
```yaml
---
- hosts: linux
  vars:
    jobids: []
  tasks:
    - name: Task 1
      command: /bin/sleep 5
      async: 10
      poll: 0
      register: result1

    - name: Task 2
      command: /bin/sleep 5
      async: 10
      poll: 0
      register: result2

    - name: Task 3
      command: /bin/sleep 5
      async: 10
      poll: 0
      register: result3

    - name: Task 4
      command: /bin/sleep 30
      async: 60
      poll: 0
      register: result4

    - name: Task 5
      command: /bin/sleep 5
      async: 10
      poll: 0
      register: result5

    - name: Task 6
      command: /bin/sleep 5
      async: 10
      poll: 0
      register: result6

    - name: Capture Job IDs
      set_fact:
        jobids: >
                {% if item.ansible_job_id is defined -%}
                  {{ jobids + [item.ansible_job_id] }}
                {% else -%}
                  {{ jobids }}
                {% endif %}
      with_items: "{{ [ result1, result2, result3, result4, result5, result6 ] }}"

    - name: Show Job IDs
      debug:
        var: jobids

    - name: 'Wait for Job IDs'
      async_status:
         jid: "{{ item }}"
      with_items: "{{ jobids }}"
      register: jobs_result
      until: jobs_result.finished
      retries: 30
...
```

(can be also run now without `when` to execute against all hosts)

By default ansible uses linear strategy, it means all hosts needs to execute given task before moving on to the next task.

We can tweak it up by changing ansible.cfg

```ini
[defaults]
inventory = hosts
host_key_checking = False
forks=6
```

**serial keyword:**

Executes tasks in batches for 2, so it allows whole process to finish quicker on specifc hosts:
```yaml
---
- hosts: linux
  gather_facts: false
  serial: 2
  tasks:
    - name: Task 1
      command: /bin/sleep 1

    - name: Task 2
      command: /bin/sleep 1

    - name: Task 3
      command: /bin/sleep 1

    - name: Task 4
      command: /bin/sleep 1

    - name: Task 5
      command: /bin/sleep 1

    - name: Task 6
      command: /bin/sleep 1

...
```

It can be also executed in the incremental stages:
```yaml
---
- hosts: linux
  gather_facts: false
  serial: 
    - 1
    - 2
    - 3
  tasks:
    - name: Task 1
      command: /bin/sleep 1

    - name: Task 2
      command: /bin/sleep 1

    - name: Task 3
      command: /bin/sleep 1

    - name: Task 4
      command: /bin/sleep 1

    - name: Task 5
      command: /bin/sleep 1

    - name: Task 6
      command: /bin/sleep 1
...
```

Or using percentages:
```yaml
---
- hosts: linux
  gather_facts: false
  serial: 
    - 16%
    - 34%
    - 50%
  tasks:
    - name: Task 1
      command: /bin/sleep 1

    - name: Task 2
      command: /bin/sleep 1

    - name: Task 3
      command: /bin/sleep 1

    - name: Task 4
      command: /bin/sleep 1

    - name: Task 5
      command: /bin/sleep 1

    - name: Task 6
      command: /bin/sleep 1
...
```

Randomly sleep up to 10 sec:
```yaml
---
- hosts: linux
  gather_facts: false
  tasks:
    - name: Task 1
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 2
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 3
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 4
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 5
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 6
      command: "/bin/sleep {{ 10 |random}}"
...
```

Same with strategy free(instead of linear strategy):
```yaml
---
- hosts: linux
  gather_facts: false
  strategy: free
  tasks:
    - name: Task 1
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 2
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 3
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 4
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 5
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 6
      command: "/bin/sleep {{ 10 |random}}"
...
```

