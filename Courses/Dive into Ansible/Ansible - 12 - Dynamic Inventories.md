>Dynamic inventories automatically generate the list of managed hosts and groups by querying external sources like cloud providers or CMDBs. They enable flexible, up-to-date host management without manually maintaining static inventory files.

- Needs to be an executable file. Can be written in any language providing that it can be executed from the command line
- Accepts the command line options of --list and --host hostname
- Returns a JSON encoded dictionary of inventory content when used with --list
- Returns a basic JSON encoded dictionary structure for --host hostname

```python
#!/usr/bin/env python3

'''
Dynamic inventory for Ansible in Python
'''

# Use print functionality from Python 3 for compatibility
from __future__ import print_function

import argparse
import logging

# Attempt to import json, if it fails, import simplejson
try:
    import json
except ImportError:
    import simplejson as json

# Inherit from object for Python 2/3 compatibility
class Inventory(object):

    # Constructor
    def __init__(self, include_hostvars_in_list):

        # Configure logger
        #self.configure_logger()

        # Capture and store include_hostvars_in_list
        self.include_hostvars_in_list = include_hostvars_in_list

        # Capture the script command line arguments
        parser = argparse.ArgumentParser()
        parser.add_argument('--list', action='store_true',
                            help='list inventory')
        parser.add_argument('--host', action='store',
                            help='show HOST variables')
        self.args = parser.parse_args()

        # If not called with --host or --list, show usage and exit
        if not (self.args.list or self.args.host):
            parser.print_usage()
            raise SystemExit

        # Capture and store the inventory
        self.define_inventory()

        # When called with --list, print the inventory
        if self.args.list:
            self.print_json(self.list())

        # If called with --host, print host information
        elif self.args.host:
            self.print_json(self.host())

    def define_inventory(self):
        self.groups = {
            "centos": {
                "hosts": ["centos1", "centos2", "centos3"],
                "vars": {
                    "ansible_user": 'root'
                }
            },
            "control": {
                "hosts": ["ubuntu-c"],
            },
            "ubuntu": {
                "hosts": ["ubuntu1", "ubuntu2", "ubuntu3"],
                "vars": {
                    "ansible_become": True,
                    "ansible_become_pass": 'password'
                }
            },
            "linux": {
                "children": ["centos", "ubuntu"],
            }}

        self.hostvars = {
            'centos1': {
                'ansible_port': 2222
            },
            'ubuntu-c': {
                'ansible_connection': 'local'
            }
        }

    # Pretty print JSON
    def print_json(self, content):
        print(json.dumps(content, indent=4, sort_keys=True))

    # Return inventory dictionary
    def list(self):

        #self.logger.info('list executed')

        # If include_hostvars_in_list is True, merge the hostvars
        # as _meta data
        if self.include_hostvars_in_list:
            merged = self.groups
            merged['_meta'] = {}
            merged['_meta']['hostvars'] = self.hostvars
            return merged

        # Otherwise, return the groups without hostvars
        else:
            return self.groups

    # Return host dictionary
    def host(self):

        #self.logger.info('host executed for {}'.format(self.args.host))

        # If the requested hosts exists in hostvars, return it
        if self.args.host in self.hostvars:
            return self.hostvars[self.args.host]

        # Otherwise, return an empty list
        else:
            return {}

    # Logger, for debugging as stdout is used by the script
    def configure_logger(self):
        self.logger = logging.getLogger('ansible_dynamic_inventory')
        self.hdlr = logging.FileHandler('/var/tmp/ansible_dynamic_inventory.log')
        self.formatter = logging.Formatter('%(asctime)s %(levelname)s %(message)s')
        self.hdlr.setFormatter(self.formatter)
        self.logger.addHandler(self.hdlr) 
        self.logger.setLevel(logging.DEBUG)

# Call the Inventory class constructor (__init__)
# Pass include_hostsvars_in_list as True to include hostvars
# as _meta data in list output
Inventory(include_hostvars_in_list=False)
~
```

`inventory.py --host centos1`

Returns:
```json
{
    "ansible_port": 2222
}
```

To run with dynamic inventory:
`ansible all -i inventory.py --list-hosts`
`ansible all -i inventory.py -m ping -o`

Speed can be improved by including host_vars in list.
It can be checked in the example 04:

```python

#!/usr/bin/env python3

'''
Dynamic inventory for Ansible in Python
'''

# Use print functionality from Python 3 for compatibility
from __future__ import print_function

import argparse
import logging

# Attempt to import json, if it fails, import simplejson
try:
    import json
except ImportError:
    import simplejson as json

# Inherit from object for Python 2/3 compatibility
class Inventory(object):

    # Constructor
    def __init__(self, include_hostvars_in_list):

        # Configure logger
        self.configure_logger()

        # Capture and store include_hostvars_in_list
        self.include_hostvars_in_list = include_hostvars_in_list

        # Capture the script command line arguments
        parser = argparse.ArgumentParser()
        parser.add_argument('--list', action='store_true',
                            help='list inventory')
        parser.add_argument('--host', action='store',
                            help='show HOST variables')
        self.args = parser.parse_args()

        # If not called with --host or --list, show usage and exit
        if not (self.args.list or self.args.host):
            parser.print_usage()
            raise SystemExit

        # Capture and store the inventory
        self.define_inventory()

        # When called with --list, print the inventory
        if self.args.list:
            self.print_json(self.list())

        # If called with --host, print host information
        elif self.args.host:
            self.print_json(self.host())

    def define_inventory(self):
        self.groups = {
            "centos": {
                "hosts": ["centos1", "centos2", "centos3"],
                "vars": {
                    "ansible_user": 'root'
                }
            },
            "control": {
                "hosts": ["ubuntu-c"],
            },
            "ubuntu": {
                "hosts": ["ubuntu1", "ubuntu2", "ubuntu3"],
                "vars": {
                    "ansible_become": True,
                    "ansible_become_pass": 'password'
                }
            },
            "fake" : {
               "hosts": ['fake1', 'fake2', 'fake3', 'fake4', 'fake5', 'fake6', 'fake7', 'fake8', 'fake9', 'fake10', 'fake11', 'fake12', 'fake13', 'fake14', 'fake15', 'fake16', 'fake17', 'fake18', 'fake19', 'fake20', 'fake21', 'fake22', 'fake23', 'fake24', 'fake25', 'fake26', 'fake27', 'fake28', 'fake29', 'fake30']
            },
            "linux": {
                "children": ["centos", "ubuntu"],
            }}

        self.hostvars = {
            'centos1': {
                'ansible_port': 2222
            },
            'ubuntu-c': {
                'ansible_connection': 'local'
            }
        }

    # Pretty print JSON
    def print_json(self, content):
        print(json.dumps(content, indent=4, sort_keys=True))

    # Return inventory dictionary
    def list(self):

        self.logger.info('list executed')

        # If include_hostvars_in_list is True, merge the hostvars
        # as _meta data
        if self.include_hostvars_in_list:
            merged = self.groups
            merged['_meta'] = {}
            merged['_meta']['hostvars'] = self.hostvars
            return merged

        # Otherwise, return the groups without hostvars
        else:
            return self.groups

    # Return host dictionary
    def host(self):

        self.logger.info('host executed for {}'.format(self.args.host))

        # If the requested hosts exists in hostvars, return it
        if self.args.host in self.hostvars:
            return self.hostvars[self.args.host]

        # Otherwise, return an empty list
        else:
            return {}

    # Logger, for debugging as stdout is used by the script
    def configure_logger(self):
        self.logger = logging.getLogger('ansible_dynamic_inventory')
        self.hdlr = logging.FileHandler('/var/tmp/ansible_dynamic_inventory.log')
        self.formatter = logging.Formatter('%(asctime)s %(levelname)s %(message)s')
        self.hdlr.setFormatter(self.formatter)
        self.logger.addHandler(self.hdlr) 
        self.logger.setLevel(logging.DEBUG)

# Call the Inventory class constructor (__init__)
# Pass include_hostsvars_in_list as True to include hostvars
# as _meta data in list output
Inventory(include_hostvars_in_list=True)
```

To check:
`./inventory.py --list`

This enhancement was added in 1.3+ and should be used because it dramatically increases speed of the execution.

If ever needs to be used, check AWS Inventories for reference:
https://docs.ansible.com/ansible/latest/collections/amazon/aws/docsite/aws_ec2_guide.html