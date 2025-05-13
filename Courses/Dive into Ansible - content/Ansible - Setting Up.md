
When logging in via `ssh ubuntu1` keys are exchanged and stored in `.ssh`

You can verify by comparing `known_hosts` with:
`ssh-keygen -H -F 172.18.0.2`
`ssh-keygen -H -F ubuntu1`

`ssh-keygen` generates pair of private/public

To copy ssh key on the server:
`ssh-copy-id ansible@ubuntu1`

`ssh-copy-id` copies public key to target server to .ssh/authorized_keys

**Automating with bash to apply on multiple machines:**

`sudo apt install sshpass`

```bash
ansible@ubuntu-c:~$ for user in ansible root
> do
>   for os in ubuntu centos
>   do
>     for instance in 1 2 3
>     do
>       sshpass -f password.txt ssh-copy-id -o StrictHostKeyChecking=no ${user}@${os}${instance}
>     done
>   done
> done
```

**Verify ansible connection to target hosts works**
`ansible -i,ubuntu1,ubuntu2,ubuntu3,centos1,centos2,centos3 all -m ping`

-i specifies invetories
-m specifies the module to execute

