# Ansible role: db-mysql

Installation of the db engine MariaDB using the official repo from mariadb.org.  
Able to support to move the DB and configuration locations. If changed, a symlink will be left at the original location, to make the move transparent.  

Allow to create databases and users.  

The role also make use of a variable scan to retrieve the declaration of users and databases in different groups from the ansible inventory.


## Restrictions and limitations

The db and user creation do not implement all the possibilities of the ansible modules for mysql.  
Thus, the supported options are limited.  


## Requirements

Ansible collections:
* community.mysql

Mandatory roles :
* _include-pkg_install
* _include-data_move_location


## Parameters

### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| mysql_version | The MariaDB version to install.<br />Do not specify the minor number.<br />Ex: "10.5" | "string" |


### Optional parameters

#### General parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| mysql_listen_address | MariaDB listen IP.<br />Specify 0.0.0.0 to listen on all IP | "string" | "localhost" |
| mysql_listen_port | MariaDB listen port.<br />While it is possible to change it, many utilities still have a hard-coded value, not allowing to change this port | "string" or numeric | "3306" |
mysql_system_log_rotate_days | Number of days to keep the system execution logs | numeric | 7 |
| |
| mysql_conf_dir | MariaDB configuration directory | "string" | "/etc/mysql" |
| mysql_data_dir | MariaDB databases directory | "string" | "/var/lib/mysql" |
| mysql_log_dir | Log directory | "string" | "/var/log/mysql" |
| mysql_move_data_both_exist_ignore_source | set to yes to ignore the source if both source and destination exist when moving the data.<br />This will usually occur when reinstalling the server. | boolean | no |
| |
| mysql_config_connection_max | Number of maximum simultaneous connections | numeric | 25 |
| mysql_config_open_files_limit | Maximum opened files | numeric | 2048 |
| mysql_config_default_collation | Default database collation.<br >Ref : http://mysql.rjweb.org/utf8_collations.html | "string" | "utf8mb4_unicode_ci" |
| mysql_config_default_charset | Default connection character set | "string" | "utf8mb4" |
| mysql_config_slow_query_log | Activate the tracking of slow SQL queries into a log file | "string" | "0" |
| mysql_config_performance_schema | MariaDB performance schema.<br />Set to "ON" to activate. | "string" | "OFF" |
| mysql_config_settings_raw |Add any settings to the MariaDB configuration file | list[ "string" ] | [ ] |


#### Database and user parameters

This works by looking for all groups a server is a member of, then retrieving all existing variables for the host with the following format : `<my_group>_suffix`  

For example, if a host is a member of the group `my_app1`, a variable named `my_app1_mysql_db` will be used as a valid source to create databases in MariaDB.  
If a server also has a variable named `my_app2_mysql_db`, but is not part of the my_app2 group, it will be ignored.


**Database creation**  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<my_group>_mysql_db` | list of databases to create | list[ object ] | [ ] |
| - | Database name | db: "string" | mandatory |
| - | Database collation | collation: "string" | "utf8mb4_unicode_ci" |
| - | connexion encoding | encoding: "string" | "utf8mb4" |


**User creation**  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<my_group>_mysql_user` | List of users to create | list[ object ] | [ ] |
| - | User login name | name: "string" | mandatory |
| - | User password | password: "string" | "" |
| - | User host or IP it is allowed to connect to.<br />Use the server name (inventory_hostname) for a remote connection | host: "string" | 'localhost' |
| - | Status of the user. Supported values : "present" or "absent"  | state: "string" | "present" |
| - | List of privileges for the user | priv: "string" | "" |
| - | The field names in the privileges will be upper-cased when set to false, which is the usual state.<br />The community.mysql collection version must be at least v3.8.0 | column_case_sensitive: boolean | omitted |
| - | Allow to append the privileges on an existing user instead of replacing them | append_privs: boolean | omitted |
| - | Same as append, but remove some privileges instead.<br />Exclusive with append_privs | subtract_privs: boolean | omitted |


## Usage examples

Install the version 10.6 and move the configuration and directories under /server/mysql
```
mysql_version: "10.6"
mysql_listen_address: "127.0.0.1"

mysql_root_dir: "/server/mysql"
mysql_conf_dir: "{{ mysql_root_dir }}/conf"
mysql_data_dir: "{{ mysql_root_dir }}/data"
mysql_log_dir: "{{ mysql_root_dir }}/log"
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

