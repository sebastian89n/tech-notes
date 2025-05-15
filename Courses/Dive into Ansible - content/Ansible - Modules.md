To execute **setup** module:
`ansible centos1 -m setup`
`ansible centos1 -m setup | more`

**Files** module sets attributes of files, symlinks and directories or removes them.
Many other modules support the same options as the **file** module, including **copy**, **template** and **assemble**. 

**File module:**
For Windows targets, use the **win_file** module instead.

`ansible all -m file -a 'path=/tmp/test state=touch'`
`ssh cenetos2 ls -althr /tmp/test`

**Colors output:**
- Red = Failure
- Yellow = Success, with Changes
- Green = Success, no Changes

**Unix permissions:**
Permission 600 = read/write on the user only. 0 on the group and other.

`ansible all -m file -a 'path=/tmp/test state=file mode=600'`

*An operation is indempotent, if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions.*

**Copy** module:
- Used for copying filees, from the local or remote, to a location on the remote. Use the **fetch** module, to copy files from a remote target, to a local target
- if you need a variable interpolation in the copied files, use the **template** module
- for Windows targets, use **win_copy** module instead

`touch /tmp/x`
`ansible all -m copy -a 'src=/tmp/x dest=/tmp/x'`

`remote_src=yes` -copy on the remote target using a remote source, to a remote destination:
`ansible all -m copy -a 'remote_src=yes src=/tmp/x dest=/tmp/y'`

**Command** module:
- default module so no need to specify it
- the `command` module, takes the command name followed by a list of space-delimited arguments
- the given command will be executed on all selected nodes
- it is not processed through the shell, so, variables like $HOME and operations like <, >, |, ; and & will not work. Use **shell** module if you need these feature
- for Windows targets, use the **win_command** module instead

`ansible all -a 'hostname' -o`
`ansible all -a 'touch /tmp/test_command_module creates=/tmp/test_command_module' -o` - gets a warning to use **file** module instead. Warnings can be disabled with `command_warnings=False`.

`ansible all -a 'rm /tmp/test_command_module removes=/tmp/test_command_module' -o`

**Fetch** module:
`ansible linux -m file -a 'path=/tmp/test_modules.txt state=touch mode=600' -o`
`ansible linux -m fetch -a 'src=/tmp/test_modules.txt dest=/tmp/test'`

**ansible-doc** can be used to get information on module
`ansible-doc file`
`ansible-doc fetch`


