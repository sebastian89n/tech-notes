#### Ansible configuration files (in priority order)
1. `ANSIBLE_CONFIG` (Environment Variable, with a filename target)
2. `./ansible.cfg` (an ansible.cfg file, in the current directory) -> **recommended**
3. `~/.ansible.cfg` (a hidden file, called .ansible.cfg, in the users home dir) 
4. `/etc/ansible/ansible.cfg` (typically provided, through package or system 

(4)

```
root@ubuntu-c:~# mkdir /etc/ansible
root@ubuntu-c:~# touch /etc/ansible/ansible.cfg
```

```
root@ubuntu-c:~# ansible --version
ansible [core 2.17.4]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```
(highlights config file)

(3) Switching to use home directory .ansible.cfg:

```
ansible@ubuntu-c:~$ cd ~
ansible@ubuntu-c:~$ touch .ansible.cfg
ansible@ubuntu-c:~$ ansible --version
ansible [core 2.17.4]
  config file = /home/ansible/.ansible.cfg
  configured module search path = ['/home/ansible/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /home/ansible/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```

(2)
```
ansible@ubuntu-c:~$ mkdir testdir
ansible@ubuntu-c:~$ cd testdir/
ansible@ubuntu-c:~/testdir$ touch ansible.cfg
ansible@ubuntu-c:~/testdir$ ansible --version
ansible [core 2.17.4]
  config file = /home/ansible/testdir/ansible.cfg
  configured module search path = ['/home/ansible/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /home/ansible/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```

(1)
```
ansible@ubuntu-c:~/testdir$ touch this_is_my_example_ansible.cfg
ansible@ubuntu-c:~/testdir$ export ANSIBLE_CONFIG=/home/ansible/testdir/this_is_my_example_ansible.cfg 
ansible@ubuntu-c:~/testdir$ ansible --version
ansible [core 2.17.4]
  config file = /home/ansible/testdir/this_is_my_example_ansible.cfg
  configured module search path = ['/home/ansible/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /home/ansible/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
ansible@ubuntu-c:
```

`UNSET ANSIBLE_CONFIG` to remove environment variable