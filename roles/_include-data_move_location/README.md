# Ansible role: _include-data_move_location


Move a given directory to another location, and create a symlink to alleviate the location change.  

Unless forced, the source data will not be moved if the target already contains existing data. Instead, an error will be raised.


## Restrictions and limitations

This role is not meant to be used as-is, but to be called by another role as a function.

Insert a call in your tasks with the following command:
```
# import the package install role
- include_role: name=_include-data_move_location
  vars:
    <parameters>
```


## Requirements

None.


## Parameters


### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| arg_move_data_comment | The task name title for each task execution in the ansible output.<br />While it can be omitted, it will be visible as a "CHANGE_ME" comment | "string" |
| arg_move_data_source_path | Source directory to move. The full path is expected | "string" |
| arg_move_data_dest_path | Destination path to move to the source directory | "string" |



### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_move_data_service_stop |  System service to stop when the data must be moved.<br />The service will not be restarted by this role | "string" | "" |
| arg_move_data_dest_owner | Owner of the destination directory.<br />The user must already exists | "string" | "root" |
| arg_move_data_dest_group | Group of the destination directory.<br />The group must already exists | "string" | "root" |
| arg_move_data_dest_owner_group_update_recursive | Propagate recursively the user and group.<br />Only apply if the destination type is a directory | boolean | no |
| arg_move_data_dest_mode | Access mode (octal) of the destination directory. | "string" | "0755" |
| arg_move_data_dest_type | Destination type | "directory" or "file" |  "directory" |
| arg_move_data_create_symlink | Create a symlink redirecting from "old location" => "new location" | boolean | yes |
| arg_move_data_both_exist_fail | Conflict: when both the source and destination exist with data inside, raise an error and halt the execution | boolean | yes |
| arg_move_data_both_exist_delete_source | Conflict : Ignore the source - rename to <source>-dist and proceed, ignoring the content<br />`arg_move_data_both_exist_fail` must also be set to `no` | boolean | no 
| arg_move_data_both_exist_delete_dest | Conflict: Ignore the destination if already existing - it will be deleted<br />`arg_move_data_both_exist_fail` must also be set to `no` | boolean | no |

While the role is meant for handling directories, moving a single file is also supported.


## Usage examples

Moving the MySQL /etc/mysql configuration directory to a dedicated filesystem under /server.

```
- name: MySQL/MariaDB | move location - config dir
  include_role: name=_include-data_move_location
  vars:
    arg_move_data_comment: "MySQL/MariaDB - conf"
    arg_move_data_source_path: "/etc/mysql"
    arg_move_data_dest_path: "/server/mysql/conf"
    arg_move_data_dest_owner: "root"
    arg_move_data_dest_group: "root"
    arg_move_data_dest_mode: "0755"
    arg_move_data_service_stop: "mysql.service"
    # the source is not really deleted but renamed to -dist
    arg_move_data_both_exist_delete_source: yes
    arg_move_data_both_exist_fail: no
```

Same with the /var/lib/mysql database directory 

```
- name: MySQL/MariaDB | move location - data dir
  include_role: name=_include-data_move_location
  vars:
    arg_move_data_comment: "MySQL/MariaDB - data"
    arg_move_data_source_path: "/var/lib/mysql"
    arg_move_data_dest_path: "/server/mysql/data"
    arg_move_data_dest_owner: "mysql"
    arg_move_data_dest_group: "mysql"
    # propagate the user:group to all files and directories under the destination
    arg_move_data_dest_owner_group_update_recursive: yes
    arg_move_data_dest_mode: "0750"
    arg_move_data_service_stop: "mysql.service"
    # no automatic action with the data, will be managed manually
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

