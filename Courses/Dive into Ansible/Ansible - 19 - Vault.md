>**Ansible Vault** is a feature in Ansible that allows you to encrypt sensitive data such as passwords, API keys, or entire configuration files, so they can be safely stored in version control. It ensures that only users with the correct password or key can decrypt and use the encrypted content during playbook execution. Vault helps secure secrets without needing a separate secrets manager.

- encrypting / decrypting variables
- encrypting / decrypting files
- re-ncrypting data
- using multiple vaults

`group_vars` / `ubuntu`
```yaml
---
ansible_become: true
ansible_become_pass: password
...
```

In order to store password securely, we can use vault.

`ansible-vault encrypt_string --ask-vault-pass --name 'ansible_become_pass' 'password'`

```bash
ansible_become_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66356461333866333562383334613230323063336336363638303835656139323931343131306139
          3836663437313431656161613463383962363037393863310a383630313464623935366131353165
          65316261666136323866363066373564313839616531633739333663323232663633373164326265
          3436633530363030330a386463643539613532363761353061666639376432316130316437323434
          6339

```

In order to use Vault:

`ansible --ask-vault-pass ubuntu -m ping -o`

---

**Encrypting file:**

`ansible-vault encrypt external_vault_vars.yaml` - this will result in file content being encrypted by ansibel

```yaml
---
- hosts: linux
  vars_files:
     - external_vault_vars.yaml
  tasks:
    - name: Show external_vault_var
      debug:
        var: external_vault_var
...
```

`ansible-playbook --ask-vault-pass vault_playbook.yaml`

**Decrypting file:**
`ansible-vault decrypt external_vault_vars.yaml`

`ansible-vault rekey external_vault_vars.yaml` - decrypt file using current vault password and re-encrypt it using new password. It is used to **rotate vault password**(for security)

`ansible-vault view external_vault_vars.yaml` - view content of the file

```bash
ansible@ubuntu-c:~/diveintoansible/Ansible Playbooks, Deep Dive/Vault/02$ echo vaultpass > password_file
ansible@ubuntu-c:~/diveintoansible/Ansible Playbooks, Deep Dive/Vault/02$ ansible-vault view --vault-password-file=password_file external_vault_vars.yaml 
```

Store password in the file and read from file when viewing.

`ansible-vault view --vault-id @prompt external_vault_vars.yaml` - more flexible way to pass passwords

`name_of_vault@[filename|prompt]`

`ansible-vault view --vault-id @password_file external_vault_vars.yaml` (no name of vault here because we did not set it up)

**Encrypting with vault-id:**
`ansible-vault encrypt --vault-id vars@prompt external_vault_vars.yaml`

`vars` is a name of the vault

To encrypt `become_password`:
```bash
ansible-vault encrypt_string --vault-id ssh@prompt --name 'ansible_become_pass' 'password'
```

We can now update value in group_vars/ubuntu with output of above command.

Now we have `external_vault_vars.yaml` encrypted with vault id `vars` and `become_password` with `ssh` id.

Playbook can be run with following command:
`ansible-playbook --vault-id vars@prompt --vault-id ssh@prompt vault_playbook.yaml`

To encrypt whole playbook:
`ansible-vault encrypt --vault-id playbook@prompt vault_playbook.yaml ``


(varspass / sshpass / playbookpass for vault pass)