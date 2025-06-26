>In Ansible, **plugins** are modular components that extend its core functionality — such as how it connects to hosts, logs output, parses inventory, or handles variables. There are many plugin types like **connection plugins**, **inventory plugins**, **callback plugins**, and **lookup plugins**, each serving a different purpose

https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html


Custom Ansible plugins can be placed in a directory (e.g. `plugins/`) and referred to in your `ansible.cfg` using the appropriate plugin path settings (e.g., `action_plugins`, `connection_plugins`, etc.). Each plugin type must be placed in a specific subfolder with a defined name. Here's the list:

**Plugin Folder Guidelines**

- Create a base folder like `plugins/`
- Inside it, add subfolders for each plugin type using the correct name
- Point to the base folder in your `ansible.cfg`, for example:

```ini
[defaults]
action_plugins = ./plugins/action
connection_plugins = ./plugins/connection
lookup_plugins = ./plugins/lookup
```

If you don’t explicitly configure plugin paths in `ansible.cfg`, Ansible will still attempt to load plugins from default folder names, relative to the current working directory (where the playbook is run).

```
your-playbook-dir/
├── playbook.yml
├── action_plugins/
├── callback_plugins/
├── connection_plugins/
├── filter_plugins/
├── inventory_plugins/
├── lookup_plugins/
├── strategy_plugins/
├── vars_plugins/
├── test_plugins/
├── shell_plugins/
├── terminal_plugins/
├── httpapi_plugins/
├── netconf_plugins/
├── cliconf_plugins/
├── cache_plugins/
```

🔌 **Ansible Plugin Types:**

Ansible uses plugins to extend its functionality. Below is a list of core plugin types with brief descriptions:

1. Connection Plugins
- Define how Ansible connects to target systems.
- Examples: `ssh`, `paramiko`, `local`, `docker`, `winrm`

2. Inventory Plugins
- Dynamically generate the list of hosts from cloud providers or other external sources.
- Examples: `aws_ec2`, `gcp_compute`, `azure_rm`, `yaml`, `ini`

3. Lookup Plugins
- Retrieve data from external sources during playbook execution.
- Examples: `file`, `env`, `passwordstore`, `pipe`, `csvfile`, `hashi_vault`

4. Callback Plugins
- Control the output of Ansible runs or send data to external systems.
- Examples: `default`, `json`, `yaml`, `minimal`, `slack`, `logstash`

5. Filter Plugins
- Used to transform data inside Jinja2 templates.
- Examples: `to_nice_json`, `json_query`, `ipaddr`, custom filters

6. Test Plugins
- Used for conditionals in Jinja2 (`when` statements), return `true` or `false`.
- Examples: `is defined`, `match`, `search`, `version_compare`

7. Vars Plugins
- Load variables from external sources or files.
- Examples: `host_group_vars`, `yaml`, `json`, `ini`, custom

8. Action Plugins
- Customize how a task is run on the controller before delegating to the remote host.
- Commonly used internally for modules.

9. Strategy Plugins
- Control the order in which tasks are executed across hosts.
- Examples: `linear`, `free`, `host_pinned`

10. Cache Plugins
- Store facts to improve performance across runs.
- Examples: `jsonfile`, `redis`, `memcached`

 11. Inventory Cache Plugins
- Specifically for caching inventory plugin results.

 12. Shell Plugins
- Define how command-line execution is handled on target systems.
- Examples: `sh`, `csh`, `powershell`

 13. Netconf Plugins
- Manage devices using the NETCONF protocol.

 14. Terminal Plugins
- Customize the CLI prompt for network devices.

 15. HTTPAPI Plugins
- Handle connections to devices using HTTP APIs (e.g., REST).

 16. CLI (Click) Plugins
- Extend the `ansible` or `ansible-galaxy` CLI with additional commands.

 17. Inventory Source Plugins
- Handle parsing of inventory sources like `inventory_dir`, `inventory_file`, etc.
---
**How to create `with_sorted_items` plugin:**

`sorted_items_playbook.yaml`
```yaml
---
- hosts: centos1
  tasks:
     - name: loop through list
       debug: 
         msg: "An item: {{item}}"
       with_sorted_items:
         - 3
         - 2
         - 1
         - Z
         - A
         - M
...
```

**Expected processing order:** 1, 2, 3, A, M, Z

```bash
ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Plugins/01$ mkdir lookup_plugins

ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Plugins/01$ cd lookup_plugins/

wget https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/plugins/lookup/items.py

ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Plugins/01/lookup_plugins$ mv items.py sorted_items.py
```

We replcae in vim: :`1,$s/with_items/with_sorted_items/g`
and update sorting in `return self._flatten(sorted(terms, key=str))`
`https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/lookup/__init__.py` - skeleton for a lookup plugin

Actual `sorted_items.py` after updates:
```python
# (c) 2012, Michael DeHaan <michael.dehaan@gmail.com>
# (c) 2017 Ansible Project
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import annotations

DOCUMENTATION = """
    name: items
    author: Michael DeHaan
    version_added: historical
    short_description: list of items
    description:
      - this lookup returns a list of items given to it, if any of the top level items is also a list it will flatten it, but it will not recurse
    notes:
      - this is the standard lookup used for loops in most examples
      - check out the 'flattened' lookup for recursive flattening
      - if you do not want flattening nor any other transformation look at the 'list' lookup.
    options:
      _terms:
        description: list of items
        required: True
"""

EXAMPLES = """
- name: "loop through list"
  ansible.builtin.debug:
    msg: "An item: {{ item }}"
  with_sorted_items:
    - 1
    - 2
    - 3

- name: add several users
  ansible.builtin.user:
    name: "{{ item }}"
    groups: "wheel"
    state: present
  with_sorted_items:
     - testuser1
     - testuser2

- name: "loop through list from a variable"
  ansible.builtin.debug:
    msg: "An item: {{ item }}"
  with_sorted_items: "{{ somelist }}"

- name: more complex items to add several users
  ansible.builtin.user:
    name: "{{ item.name }}"
    uid: "{{ item.uid }}"
    groups: "{{ item.groups }}"
    state: present
  with_sorted_items:
     - { name: testuser1, uid: 1002, groups: "wheel, staff" }
     - { name: testuser2, uid: 1003, groups: staff }

"""

RETURN = """
  _raw:
    description:
      - once flattened list
    type: list
"""

from ansible.plugins.lookup import LookupBase


class LookupModule(LookupBase):

    def run(self, terms, variables=None, **kwargs):

        return self._flatten(sorted(terms, key=str))
```

---
**Another example with customized filter:**
```bash
ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Plugins/02$ mkdir filters_plugins

ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Plugins/02$ cd filters_plugins/

ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Plugins/02/filters_plugins$ wget https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/plugins/filter/core.py

ansible@ubuntu-c:~/diveintoansible/Creating Modules and Plugins/Creating Plugins/02/filters_plugins$ mv core.py reverse_upper.py
```

In vim: 
- to show numbers: `:set number`
- to delete lines `:64,192d`

+/- stripped and edited python code in `reverse_upper.py`:
```python
# (c) 2012, Jeroen Hoekx <jeroen@hoekx.be>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)

from __future__ import annotations

import base64
import functools
import glob
import hashlib
import json
import ntpath
import os.path
import re
import shlex
import time
import uuid
import yaml
import datetime
import typing as t

from collections.abc import Mapping
from functools import partial
from random import Random, SystemRandom, shuffle

from jinja2.filters import do_map, do_select, do_selectattr, do_reject, do_rejectattr, pass_environment, sync_do_groupby
from jinja2.environment import Environment

from ansible._internal._templating import _lazy_containers
from ansible.errors import AnsibleFilterError, AnsibleTypeError, AnsibleTemplatePluginError
from ansible.module_utils.datatag import native_type_name
from ansible.module_utils.common.json import get_encoder, get_decoder
from ansible.module_utils.six import string_types, integer_types, text_type
from ansible.module_utils.common.text.converters import to_bytes, to_native, to_text
from ansible.module_utils.common.collections import is_sequence
from ansible.parsing.yaml.dumper import AnsibleDumper
from ansible.template import accept_args_markers, accept_lazy_markers
from ansible._internal._templating._jinja_common import MarkerError, UndefinedMarker, validate_arg_type
from ansible._internal._yaml import _loader as _yaml_loader
from ansible.utils.display import Display
from ansible.utils.encrypt import do_encrypt, PASSLIB_AVAILABLE
from ansible.utils.hashing import md5s, checksum_s
from ansible.utils.unicode import unicode_wrap
from ansible.utils.vars import merge_hash

display = Display()

UUID_NAMESPACE_ANSIBLE = uuid.UUID('361E6D51-FAEC-444A-9079-341386DA8E2E')

def reverse_upper(string):
    '''Reverse and upper string'''
    return string[::-1].upper()

class FilterModule(object):
    """ Ansible core jinja2 filters """

    def filters(self):
        return {
            'reverse_upper': reverse_upper
        }

```

`reverse_upper_filter_playbook.yaml`
```yaml
---
- hosts: all
  tasks:
     - name: Reverse and upper ansible_distribution
       debug: 
         msg: "Reverse and upper of ansible_distribution: {{ ansible_distribution | reverse_upper }}"
...
```

