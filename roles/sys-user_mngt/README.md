# Ansible role: sys-user_mngt

This role will create or remove any user or group using different possible variables.  
Pushing ssh keys and inserting sudoers configuration is supported.

This role has the capability to search in all inventory, looking for a particular variable name based on group membership's, following this format: `<one_group>_users_account`.  
As such, it is possible to declare users in multiple locations, global for all hosts, or only for a specific group or host.

Example: a host member of the inventory groups "group1" and "group2" will be scanned for variables named :
* group1_users_account
* group2_users_account

If a variable `group_test_users_account` exists for this host, it will be ignored if the host is not a member of group_test.


## Restrictions and limitations

There is no filtering applied to handle duplicates. It is possible to have the same user declared and applied multiple times for one host.  
In such case, ansible will create the user then update it to match each definition.


## Requirements

The sudo command must be installed on target hosts.


## Parameters

The following variables can be defined in groups, or at the host level.  
The usual usage is to set those in `group_vars/`.  You can then override (replacing, not merging) some of them in `host_vars/<host>` if required.


### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| users_default_shell | The default shell for a user if none is specified. | string | "/bin/bash" |
| users_default_sudo_timeout | Timeout duration for sudo - in mins | number or string | 15 |
| users_default_group | Default primary group for new users if none is specified | string | "users" |
| user_mngt_sync_ssh_keys | Allow to sync a whole ssh_keys directory to the host | boolean | false |
| user_mngt_sync_ssh_keys_ansible_source | the source directory on the ansible server to sync to the servers | string | "/opt/ansible/data/ssh_pub_keys" |
| user_mngt_sync_ssh_keys_dir_dest | The full path destination on the servers | string | "/opt/system/ssh" |

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| groups_to_create_all | List of system groups to create on all servers.<br />Example: ` - { name: "devel", gid: 10000 }`<br />  | list:<br/>- { name, gid }| [ ] |
| - | username to create  | name: "string" | |
| - | guid for the user to create  | guid: number | |

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| users_to_delete_all | Lists of users to delete on all hosts | list: [ { dict } ] | [ ] |
| - | user description, only for log output | name: "string" | mandatory |
| - | user name as existing on the system | username: "string" | mandatory |
| - | must be explicitly set to yes to remove the user | remove: boolean | no |
| - | force the homedir deletion | force: boolean | no |
| - | restrict the user removal to the servers in this list<br />Example: [ srv1, srv2, ... ] | target_hosts: [ "string" ] | [ "all" ] |
| - | skip the user removal for the servers in this list | skip_hosts: [ "string" ] | [ ] |

Both "target_hosts:" and "skip_hosts:" can also specify a group (or multiples) using `skip_hosts: "{{ groups['my_group'] + groups[...] }}"`

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
|users_accounts_all | list of users to create on all hosts | list: [ { dict } ] | [ ] |
| - | user description, will reuse the username if empty | name: "string" | "" |
| - | user system login - mandatory | username: "string" | mandatory |
| - | user system id, if not provided, the system will decide itself | uid: number | |
| - | reuse the username for the primary group name | username_as_group: boolean | false |
| - | primary user group if username_as_group=false and not wanting the default user group | group: "string" | "" |
| - | group uid, default to uid if username_as_group=true | gid: number | "{{ thisUser.uid }}" |
| - | other groups to apply to the user | groups: ['string' ] | [ "" ] |
| - | set to 'yes'  to keep both the primariy group and additional groups | groups_append: boolean | no |
| - | shell for the user | shell: "string" | "{{ users_default_shell }}" |
| - | mark the user as a system user | system_user: boolean | no |
| - | user password, managed by the system if empty (locked usually)<br />The provided password is not in clear, but already crypted | password: "string" | "" |
| - | when to update the password if changed<br />use "always" to update the password if it has been changed on the server | update_password: "on_create" or "always" | "on_create" |
| - | lock (or unlock) the user and change the password to '!' <br />Login with a ssh key or `sudo su` is still possible | password_lock: boolean | no |
| - | create the user homedir | create_homedir: boolean | no |
| - | define the user homedir | homedir: "string" | "/home/{{ thisUser.username }}" |
| - | set an expiration date to the account (epoch date format) | expires: number (epoch) | - |
| - | Use the local commands, for when a centralized authentication is present, like ldap | local: boolean | no |
| - | skip the user (and group) creation.<br /> If active with 'local', the role will still create the homedir when missing | skip_create_user: boolean | no |
| - | set to a "/path/to/local/template_name.j2" to have it copied as "/etc/sudoers.d/template_name" | sudoers: "string" | "" |
| - | create a ssh private key for the user if none is present | generate_ssh_key: boolean | no |
| - | use the text as a passphrase if a new ssh private key is created<br />The passphrase is in clear text. | ssh_key_passphrase: "string" | "" |
| - | text in the ssh public key comment area when creating a new ssh key | ssh_key_comment: "string" | "{{ thisUser.name + '@' + inventory_hostname }}" |
| - | ssh keys list to add to the user authorized_keys file.<br />Example:<br />`- "ssh-rsa AAAA.../WIf6kd throwaway@example.com"`<br />`- "ssh- ..."`<br />`- "{{ lookup('file', '/path/in/ansible/to/file.pub' ) }}"` | ssh_pub_keys: [ "string" ] | [ ] |
| - | If set to 'yes', will only allow for this user the keys listed in ssh_pub_keys and remove all others<br />If ssh_pub_keys is empty, all the user keys will be removed from the authorized_keys file | exclusive_keys: boolean | no |
| - | restrict the user creation to the servers in this list<br />Example: [ srv1, srv2, ... ] | target_hosts: [ "string" ] | [ "all" ] |
| - | skip the user creation for the servers in this list | skip_hosts: [ "string" ] | [ ] |


On linux, a user password is expected to be encrypted, use this interactive command to generate the hash : `mkpasswd --method=sha-512`  
It is provided by the package 'mkpasswd' on RHEL, 'whois' on Debian

Ansible user module's documentation and behavior : http://docs.ansible.com/ansible/latest/modules/user_module.html


**Users for specific group:**

The role supports the creation of users depending of being member of an ansible group.  
A search function is used by the role capturing all `<inventory_group>_users_account` and merging them for each related host.  
This behavior also allows to declare users only in dedicated groups without the need to select the hosts.

While it is not recommended, it is possible to create a `<inventory_group>_users_account` variable in a different group vars.  
If a host is a member of both inventory groups, the users of this declaration will be applied.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
|`<group_name>_users_account` | list of users to create on the hosts member of the group | list: [ { dict } ] | [ ] |
| | Expected parameters are the same as `users_accounts_all` |


**Users for specific server:**

This variant is expected to be placed under the `host_vars/<my host>` declarations.  
When placed on a host, it is equivalent to `target_hosts: [ "specific host" ]`

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| users_accounts_host | list of users to create on the host | list: [ { dict } ] | [ ] |
| | Expected parameters are the same as `users_accounts_all` |


## Usage examples

**Declare users in specific groups in the ansible inventory**  
The structure must be placed in `group_vars/<inventory_group>` and named as `<inventory_group>_users_account`  
Such structure can be declared in multiple groups, as long as the variable name and user declaration are uniques.

```
# Example for a 'group_vars/backup' inventory group.
# assuming there are 2 groups :
#     backup        (combining all the servers, including the group 'backup_hosts')
#     backup_hosts  (only the servers storing the backups)

backup_users_account:
  - name: "backup admin"
    username: backup
    ...
    target_hosts:  "{{ groups['backup_hosts'] }}"

  # create as a system user without password
  - name: "backup user"
    username: backup
    ...
    system_user: yes
    password_lock: yes
    ssh_pub_keys:
     - "{{ lookup('file', '/etc/ansible/storage/ssh_keys/backup.pub' ) }}"
    exclusive_keys: yes
    skip_hosts:  "{{ groups['backup_hosts'] }}"
```


**Users for specific host:**

The following declarations will allow to create users only for one host.

```
# user accounts for specific servers
# must be declared in host_vars/<host>
# with the name: users_accounts_host
# Example:
users_accounts_host:
  - name: Test_User
    username: testuser
    ... same parameters as users_accounts_all

  - name: "Another user"
    username: another
    ...
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

