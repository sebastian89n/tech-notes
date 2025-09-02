> Inventories in Ansible define the hosts and groups of hosts that playbooks manage. They can be static (INI, YAML) or dynamic (via scripts or plugins), and include variables that control host-specific behavior.

`ansible all -m ping` execute ping module on all hosts
or `ansible "*" -m ping`

Or target specific hosts group
`ansible hostGroupName -m ping`

Or specific host
`ansible host -m ping`

Output in one-liners:
`ansible all -m ping -o`
`ansible centos1 -m ping -o`

`-o` is `one-line` makes theoutput more concise by printing each result on a single line, instead of in the usual multi-line, formatted output

To disable ssh key check:
`ANSIBLE_HOST_KEY_CHECKING=False` 

To run in one-line(testing purposes only):
`ANSIBLE_HOST_KEY_CHECKING=False ansible all -m ping`

To actually disable it in ansible.cfg:
```ini
[defaults]
inventory = hosts
host_key_checking = False
```

List all hosts within a group
`ansible hostGroupName --list-hosts`
or
`ansible all --list-hosts`

To use regular expression for host name use tylda
`ansible ~.*3 -m ping -o` - ping every host that ends with 3

To run module command that will call "id" command on host env in order to display used user we can use:
`ansible all -m command -a 'id' -o` 

Command module is actually the default one so we can use also:
`ansible all -a 'id' -o` 

Users used by ansible on different hosts can be configured in hosts file, e.g.:
```ini
[centos]
centos1 ansible_user=root
centos2 ansible_user=root
centos3 ansible_user=root

[ubuntu]
ubuntu1
ubuntu2
ubuntu3
```

```ini
[centos]
centos1 ansible_user=root ansible_port=2222
centos2 ansible_user=root
centos3 ansible_user=root

[ubuntu]
ubuntu1 ansible_become=true ansible_become_pass=password
ubuntu2 ansible_become=true ansible_become_pass=password
ubuntu3 ansible_become=true ansible_become_pass=passwordtu
```

In the example above centos is configured via root, while ubuntu connect via "ansible" user that becomes a superuser. Also port on centos1 is configured to 2222.

To change ssh port on target host
`vim /etc/ssh/sshd_config`
and change `Port` variable. You can configure more than one port.

Bash script to update it:
```bash
# Create a timestamped backup of the original sshd_config file
timestamp=$(date +"%Y%m%d%H%M%S")
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak_$timestamp
echo -e "🔒 Backup of sshd_config created at /etc/ssh/sshd_config.bak_$timestamp"

# Remove existing 'Port' entries from the config file
sed -i '/^Port /d' /etc/ssh/sshd_config
echo -e "🧹 Existing Port entries removed."

# Add new port entries from the script arguments
for port in "$@"; do
  echo "Port $port" >> /etc/ssh/sshd_config
done
echo -e "🔧 New Port entries added for: $*"

# Restart the SSHD service to apply changes
if systemctl restart sshd.service; then
  echo -e "✅ SSHD has been successfully restarted and is now listening on ports: $*"
else
  echo -e "❌ Failed to restart SSHD. Check your configuration."
fi
update_sshd_ports.sh (END)

```

Using ranges for servers and defining local ubuntu-c host:
**Note: it has `ansible_connection=local` because it applies on the local machine**
```ini
[control]
ubuntu-c ansible_connection=local

[centos]
centos1 ansible_user=root ansible_port=2222
centos[2:3] ansible_user=root

[ubuntu]
ubuntu[1:3] ansible_become=true ansible_become_pass=password
```

Defining hosts vars:
```ini
[control]
ubuntu-c ansible_connection=local

[centos]
centos1 ansible_port=2222
centos[2:3]

[centos:vars]
ansible_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars]
ansible_become=true
ansible_become_pass=password
```

Defining all vars and hosts group with another hosts group as childrene:
```ini
[control]
ubuntu-c ansible_connection=local

[centos]
centos1 ansible_port=2222
centos[2:3]

[centos:vars]
ansible_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars]
ansible_become=true
ansible_become_pass=password

[linux:children]
centos
ubuntu

[linux:vars]
# ansible_port=1234

[all:vars]
# ansible_port=1234
```

To use yaml, ansible.cfg:
```ini
[defaults]
inventory = hosts.yaml
host_key_checking = False
```

hosts.yaml:
```yaml
---
control:
  hosts:
    ubuntu-c:
      ansible_connection: local
centos:
  hosts:
    centos1:
      ansible_port: 2222
    centos2:
    centos3:
  vars:
      ansible_user: root
ubuntu:
  hosts:
    ubuntu1:
    ubuntu2:
    ubuntu3:
  vars:
    ansible_become: true
    ansible_become_pass: password
linux:
  children:
    centos:
    ubuntu:
...
```

or json(not recommended):
```json
{
    "control": {
        "hosts": {
            "ubuntu-c": {
                "ansible_connection": "local"
            }
        }
    },
    "centos": {
        "hosts": {
            "centos1": {
                "ansible_port": 2222
            },
            "centos2": null,
            "centos3": null
        },
        "vars": {
            "ansible_user": "root"
        }
    },
    "ubuntu": {
        "hosts": {
            "ubuntu1": null,
            "ubuntu2": null,
            "ubuntu3": null
        },
        "vars": {
            "ansible_become": true,
            "ansible_become_pass": "password"
        }
    },
    "linux": {
        "children": {
            "centos": null,
            "ubuntu": null
        }
    }
}
```

You can specify which hosts file to use by using -i:
`ansible all -i hosts.yaml -m ping -o`

You can also specify extra vars by using:
`ansible linux -i hosts.yaml --extra-vars 'ansible_port=2222' -m ping -o`
or
`ansible linux -i hosts.yaml -e 'ansible_port=2222' -m ping -o`