- SSH connectivity
- Syntax Check
- Step
- Start At
- Log Path
- Verbosity
---
**Investigating problems with SSH:**
`ssh -v` to debug ssh access problems
```bash
ansible@ubuntu-c:~$ ssh -v ubuntu1

OpenSSH_8.9p1 Ubuntu-3ubuntu0.4, OpenSSL 3.0.2 15 Mar 2022
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: include /etc/ssh/ssh_config.d/*.conf matched no files
debug1: /etc/ssh/ssh_config line 21: Applying options for *
debug1: Connecting to ubuntu1 [172.18.0.8] port 22.
debug1: Connection established.
debug1: identity file /home/ansible/.ssh/id_rsa type 0
debug1: identity file /home/ansible/.ssh/id_rsa-cert type -1
debug1: identity file /home/ansible/.ssh/id_ecdsa type -1
debug1: identity file /home/ansible/.ssh/id_ecdsa-cert type -1
debug1: identity file /home/ansible/.ssh/id_ecdsa_sk type -1
debug1: identity file /home/ansible/.ssh/id_ecdsa_sk-cert type -1
debug1: identity file /home/ansible/.ssh/id_ed25519 type -1
debug1: identity file /home/ansible/.ssh/id_ed25519-cert type -1
debug1: identity file /home/ansible/.ssh/id_ed25519_sk type -1
debug1: identity file /home/ansible/.ssh/id_ed25519_sk-cert type -1
debug1: identity file /home/ansible/.ssh/id_xmss type -1
debug1: identity file /home/ansible/.ssh/id_xmss-cert type -1
debug1: identity file /home/ansible/.ssh/id_dsa type -1
debug1: identity file /home/ansible/.ssh/id_dsa-cert type -1
debug1: Local version string SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4
debug1: Remote protocol version 2.0, remote software version OpenSSH_8.9p1 Ubuntu-3ubuntu0.4
debug1: compat_banner: match: OpenSSH_8.9p1 Ubuntu-3ubuntu0.4 pat OpenSSH* compat 0x04000000
debug1: Authenticating to ubuntu1:22 as 'ansible'
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: algorithm: curve25519-sha256
debug1: kex: host key algorithm: ecdsa-sha2-nistp256
debug1: kex: server->client cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: kex: client->server cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: SSH2_MSG_KEX_ECDH_REPLY received
debug1: Server host key: ecdsa-sha2-nistp256 SHA256:z96FbonokCWqE89m5pGU/ug7BgdBaJbGeCivWqxdLvE
debug1: load_hostkeys: fopen /home/ansible/.ssh/known_hosts2: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts2: No such file or directory
debug1: load_hostkeys: fopen /home/ansible/.ssh/known_hosts2: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts2: No such file or directory
debug1: Host 'ubuntu1' is known and matches the ECDSA host key.
debug1: Found key in /home/ansible/.ssh/known_hosts:6
Warning: the ECDSA host key for 'ubuntu1' differs from the key for the IP address '172.18.0.8'
Offending key for IP in /home/ansible/.ssh/known_hosts:20
Matching host key in /home/ansible/.ssh/known_hosts:6
```

Ssh debug port:
`root@ubuntu1:~# /usr/sbin/sshd -d -p 1234`

Next if we want to connect to it from ubuntu-c:
`ansible@ubuntu-c:~$ ssh ubuntu1 -p 1234`

We will get useful debug information on the root terminal side.

We may get information like:
`Authentication refused: bad ownership or modes for files /home/ansible/.ssh/authorized_keys`

---
**Playbooks:**
`ansible-playbook blocks_playbook.yaml --syntax-check` - checks syntax in the playbook

`ansible-playbook blocks_playbook.yaml --step` - go through playbook step by step

`ansible-playbook blocks_playbook.yaml --start-at-task='Install python-dnspython'` - starts at specific task

In ansible.cfg we can enable execution logging by specifying `log_path`:
```ini
[defaults]
inventory = hosts
host_key_checking = False
forks=6
log_path=log.txt
```

---

**Verbose options(1-4):**
-v or -vv or -vvv or -vvvv