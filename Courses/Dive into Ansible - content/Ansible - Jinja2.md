**IF example**
```
---
-
  tasks:
    - name: Ansible Jinja2 if elif
      debug:
        msg: >
             --== Ansible Jinja2 if elif statement ==--

			 {# If the hostname is ubuntu-c, include a message -#}
             {% if ansible_hostname == "ubuntu-c" -%}
                This is ubuntu-c
             {% elif ansible_hostname == "centos1" -%}
                This is centos1 with it's modified SSH Port
             {% else -%}
                This is good old {{ ansible_hostname }}
             {% endif %}
...
```

\- before % is to prevent carrage return

**is defined example:**
```yaml
---
-
  hosts: all
  tasks:
    - name: Ansible Jinja2 if variable is defined ( where variable is not defined )
      debug:
        msg: >
             --== Ansible Jinja2 if variable is defined ( where variable is not defined ) ==--

             {% if example_variable is defined -%}
                example_variable is defined
             {% else -%}
                example_variable is not defined
             {% endif %}

...
```

**set variable example:**
```yaml
---
-
  hosts: all
  tasks:
    - name: Ansible Jinja2 if variable is defined ( where variable is defined )
      debug:
        msg: >
             --== Ansible Jinja2 if variable is defined ( where variable is defined ) ==--

             {% set example_variable = 'defined' -%}
             {% if example_variable is defined -%}
                example_variable is defined
             {% else -%}
                example_variable is not defined
             {% endif %}

...
```

**for loop example:**

```yaml
---
-
  hosts: all
  tasks:
    - name: Ansible Jinja2 for statement
      debug:
        msg: >
             --== Ansible Jinja2 for statement ==--

             {% for entry in ansible_interfaces -%}
                Interface entry {{ loop.index }} = {{ entry }}
             {% endfor %}
...
```

**for in range example(end eclusive):**
```yaml
---
-
  hosts: all
  tasks:
    - name: Ansible Jinja2 for range
      debug:
        msg: >
             --== Ansible Jinja2 for range

             {% for entry in range(1, 11) -%}
                {{ entry }}
             {% endfor %}

...
```

**break example**:
To enable extensions in ansible.cfg add:
```ini
[defaults]
inventory = hosts
host_key_checking = False
jinja2_extensions = jinja2.ext.loopcontrols
```

```yaml
---
-
  hosts: all
  tasks:
    - name: Ansible Jinja2 for range, reversed (simulate while greater 5)
      debug:
        msg: >
             --== Ansible Jinja2 for range, reversed (simulate while greater 5) ==--

             {% for entry in range(10, 0, -1) -%}
                {% if entry == 5 -%}
                   {% break %}
                {% endif -%}
                {{ entry }}
             {% endfor %}
...
```

**is odd and continue example:**

```yaml
---
-
  hosts: all
  tasks:
    - name: Ansible Jinja2 for range, reversed (continue if odd)
      debug:
        msg: >
             --== Ansible Jinja2 for range, reversed (continue if odd) ==--

             {% for entry in range(10, 0, -1) -%}
                {% if entry is odd -%}
                   {% continue %}
                {% endif -%}
                {{ entry }}
             {% endfor %}
...
```

**Filters:**
More: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html 

```yaml
---
-
  hosts: all
  tasks:
    - name: Ansible Jinja2 filters
      debug:
        msg: >
             ---=== Ansible Jinja2 filters ===---

             --== min [1, 2, 3, 4, 5] ==--

             {{ [1, 2, 3, 4, 5] | min }}

             --== max [1, 2, 3, 4, 5] ==--

             {{ [1, 2, 3, 4, 5] | max }}

             --== unique [1, 1, 2, 2, 3, 3, 4, 4, 5, 5] ==--

             {{ [1, 1, 2, 2, 3, 3, 4, 4, 5, 5] | unique }}

             --== difference [1, 2, 3, 4, 5] vs [2, 3, 4] ==--

             {{ [1, 2, 3, 4, 5] | difference([2, 3, 4]) }}

             --== random ['rod', 'jane', 'freddy'] ==--

             {{ ['rod', 'jane', 'freddy'] | random }}

             --== urlsplit hostname ==--

             {{ "http://docs.ansible.com/ansible/latest/playbooks_filters.html" | urlsplit('hostname') }}
...
```

**Template example:**

```yaml
---
-
  hosts: all
  tasks:
    - name: Jinja2 template
      template:
        src: template.j2
        dest: "/tmp/{{ ansible_hostname }}_template.out"
        trim_blocks: true
        mode: 0644
...
```

template.j2
```yaml
--== Ansible Jinja2 if statement ==--

{# If the hostname is ubuntu-c, include a message -#}
{% if ansible_hostname == "ubuntu-c" -%}
      This is ubuntu-c
{% endif %}

--== Ansible Jinja2 if elif statement ==--

{% if ansible_hostname == "ubuntu-c" -%}
   This is ubuntu-c
{% elif ansible_hostname == "centos1" -%}
   This is centos1 with it's modified SSH Port
{% endif %}

--== Ansible Jinja2 if elif else statement ==--

{% if ansible_hostname == "ubuntu-c" -%}
   This is ubuntu-c
{% elif ansible_hostname == "centos1" -%}
   This is centos1 with it's modified SSH Port
{% else -%}
   This is good old {{ ansible_hostname }}
{% endif %}

--== Ansible Jinja2 if variable is defined ( where variable is not defined ) ==--

{% if example_variable is defined -%}
   example_variable is defined
{% else -%}
   example_variable is not defined
{% endif %}

--== Ansible Jinja2 if varible is defined ( where variable is defined ) ==--

{% set example_variable = 'defined' -%}
{% if example_variable is defined -%}
   example_variable is defined
{% else -%}
   example_variable is not defined
{% endif %}

--== Ansible Jinja2 for statement ==--

{% for entry in ansible_all_ipv4_addresses -%}
   IP Address entry {{ loop.index }} = {{ entry }}
{% endfor %}

--== Ansible Jinja2 for range

{% for entry in range(1, 11) -%}
   {{ entry }}
{% endfor %}

--== Ansible Jinja2 for range, reversed (simulate while greater 5) ==--

{% for entry in range(10, 0, -1) -%}
   {% if entry == 5 -%}
      {% break %}
   {% endif -%}
   {{ entry }}
{% endfor %}

--== Ansible Jinja2 for range, reversed (continue if odd) ==--

{% for entry in range(10, 0, -1) -%}
   {% if entry is odd -%}
      {% continue %}
   {% endif -%}
   {{ entry }}
{% endfor %}

---=== Ansible Jinja2 filters ===---

--== min [1, 2, 3, 4, 5] ==--

{{ [1, 2, 3, 4, 5] | min }}

--== max [1, 2, 3, 4, 5] ==--

{{ [1, 2, 3, 4, 5] | max }}

--== unique [1, 1, 2, 2, 3, 3, 4, 4, 5, 5] ==--

{{ [1, 1, 2, 2, 3, 3, 4, 4, 5, 5] | unique }}

--== difference [1, 2, 3, 4, 5] vs [2, 3, 4] ==--

{{ [1, 2, 3, 4, 5] | difference([2, 3, 4]) }}

--== random ['rod', 'jane', 'freddy'] ==--

{{ ['rod', 'jane', 'freddy'] | random }}

--== urlsplit hostname ==--

{{ "http://docs.ansible.com/ansible/latest/playbooks_filters.html" | urlsplit('hostname') }}

```