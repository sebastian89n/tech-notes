>In Ansible, **`include`** and **`import`** are directives used to reuse and organize code by bringing in other playbooks or task files. 
>
>The key difference is that **`import_*`** (like `import_tasks`, `import_playbook`) happens at **parse time**, meaning it's static and always executed, while **`include_*`** (like `include_tasks`) occurs at **runtime**, allowing dynamic behavior such as conditional includes. 
>
>Use `import_` when the structure is fixed and known ahead of time, and `include_` when you need flexibility based on conditions or variables.

**Include example:**
```yaml
---
- hosts: all
  tasks:

     - name: Play 1 - Task 1
       debug: 
         msg: Play 1 - Task 1

     - include_tasks: play1_task2.yaml

...
```

`play1_task2.yaml`
```yaml
---
- name: Play 1 - Task 2
  debug: 
    msg: Play 1 - Task 2
...
```

**Import example:**
```yaml
---
- hosts: all
  tasks:

     - name: Play 1 - Task 1
       debug: 
         msg: Play 1 - Task 1

     - import_tasks: play1_task2.yaml
...
```

**Include is best used with when:**
```yaml
---
- hosts: centos1
  tasks:

     - debug:
         msg: ===================== Testing include_tasks =====================

     # include_tasks is dynamic
     #
     # The when statement is executed once, if the condition is met, all
     # tasks are executed
     - include_tasks: include_tasks.yaml
       when: include_tasks_var is not defined

     - debug:
         msg: ===================== Testing import_tasks ======================

     # import_tasks is static
     #
     # Each task that in the import will be independently executed against
     # the when condition
     - import_tasks: import_tasks.yaml
       when: import_tasks_var is not defined

...
```
(usage of both to demonstrage difference in behavior)

`include_tasks.yaml`
```yaml
---
- set_fact:
    include_tasks_var: foo

- name: 2nd Task
  debug: 
    msg: 2nd Task

- name: 3rd Task
  debug: 
    msg: 3rd Task
...
```

`import_tasks.yaml`
```yaml
---
- set_fact:
    import_tasks_var: foo

- name: 2nd Task
  debug: 
    msg: 2nd Task

- name: 3rd Task
  debug: 
    msg: 3rd Task
...
```

**Importing playbook:**
Importing playbook is static(not dynamic, like include).

```yaml
---
- import_playbook: imported_playbook.yaml
  when: import_playbook_var is not defined
...
```

`imported_playbook.yaml`
```yaml
---
- hosts: centos1
  tasks:

     - set_fact:
         import_playbook_var: true

     - debug:
         msg: Playbook executed
...
```

**For static** each task is independently executed against the when condition. So first task will get executed, but second one won't be executed because variable is set and when second tasks is attempted in when variable is already defined. 