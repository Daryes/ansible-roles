# Ansible role: db-postgresql

Installation of the db engine PostgreSQL using the official packages from postgresql.org.  
Able to support to move the DB and configuration locations. If changed, a symlink will be left at the original location, to make the move transparent.  

Allow to create databases and users.  

The role also make use of a variable scan to retrieve the declaration of users and databases in different groups from the ansible inventory.


## Restrictions and limitations

The db and user creation do not implement all the possibilities of the ansible modules for PostgreSQL.  
Thus, the supported options are limited.  


## Requirements

Ansible collections:
* community.postgresql

Mandatory roles :
* _include-pkg_install
* _include-data_move_location


## Parameters

### Mandatory parameters


| Parameter | Description | Type | 
| --------- | ----------- | ---- | 
| postgresql_version | The PostgreSQL major version to install.<br />Ex: "15" | "string" |


### Optional parameters

#### General parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| postgresql_listen_address | PostgreSQL listen IP.<br />Specify 0.0.0.0 to listen on all IP | "string" | "localhost" |
| postgresql_listen_port | PostgreSQL listen port.<br />While it is possible to change it, many utilities still have a hard-coded value, not allowing to change this port | "string" or numeric | "5432" |
| |
| postgresql_conf_dir | PostgreSQL configuration directory | "string" | "/etc/postgresql" |
| postgresql_data_dir | PostgreSQL databases directory | "string" | "/var/lib/postgresql" for debian<br />"/var/lib/pgsql" for rhel |
| postgresql_log_dir | Log directory | "string" | "/var/log/postgresql" |
| postgresql_move_data_both_exist_ignore_source | set to yes to ignore the source if both source and destination exist when moving the data.<br />This will usually occur when reinstalling the server. | boolean | no |
| |
| postgresql_listen_allowed_range_v4 | remote IPv4 ranges allowed to connect.<br />Only used if listen_address is not set on localhost | "string" | "0.0.0.0/0" |
| postgresql_listen_allowed_range_v6 | Same for IPv6 | "string" | "::/0"
| postgresql_listen_auth_crypt_method | Crypto method used for authentication.<br />Previous method was "md5" | "string" | "scram-sha-256" |
| postgresql_config_settings_raw |Add any settings to the PostgreSQL configuration file.<br />Ref: https://www.postgresql.org/docs/current/runtime-config.html | list[ "string" ] | [ ] |


#### Database and user parameters

This works by looking for all groups a server is a member of, then retrieve all existing variables for the host with the following format : `<my_group>_suffix`  

For example, if a host is a member of the group `my_app1`, a variable named `my_app1_postgresql_db` will be used as a valid source to create databases in PostgreSQL.  
If a server also has a variable named `my_app2_postgresql_db`, but is not part of the my_app2 group, it will be ignored.

**Database creation**  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<my_group>_postgresql_db` | list of databases to create | list[ object ] | [ ] |
| - | Database name | db: "string" | mandatory |
| - | Database encoding | encoding: "string" | "UTF8" |
| - | Collation order | lc_collate: "string" | omitted |
| - | Character classification | lc_ctype auto
| - | Tablespace name | tablespace: "string" | "pg_default" |
| - | concurrent connection limit to the db.<br />The value -1 stands for illimited | conn_limit: numeric | -1 |


**User creation**  

Ref: https://docs.ansible.com/ansible/latest/collections/community/postgresql/postgresql_user_module.html  
Ref: https://docs.ansible.com/ansible/latest/collections/community/postgresql/postgresql_privs_module.html  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<my_group>_postgresql_user` | List of users to create | list[ object ] | [ ] |
| - | User login name | name: "string" | mandatory |
| - | User password | password: "string" | "" |
| - | Expiration date, default is infinite | expires: "string" | "infinity" |
| - | User state.<br />Supported values are "present" (created) or "absent" (deleted) | state: "string" | "present" |
| - | database to apply the privileges for the user | db: "string" | "" |
| - | schema to apply the privileges for the user on the database.<br />It omitted, the privileges will be applied only on the db. | schema: "string" | omit |
| - | User privileges for both database and schema | privs: "string" | omit |

Notice: the parameters schema and privs are optional. If privs is not set, no privileges will be applied.  
Otherwise, the same privileges will be applied on the db AND the schema.  
The role is meant to give a minimum, but not handle all PostgreSQL user's subtleties.


## Usage examples

Install the version 14 and move the configuration and directories under /server/postgre
```
postgresql_version: "14"
postgresql_listen_address: "127.0.0.1"

postgresql_root_dir: "/server/postgre"
postgresql_conf_dir: "{{ postgresql_root_dir }}/conf"
postgresql_data_dir: "{{ postgresql_root_dir }}/data"
postgresql_log_dir: "{{ postgresql_root_dir }}/log"

# assuming the server is a member of the group "my_app"
# create a db name app with default settings
# and create the user appuser with "all" privileges on said db and the schema 'public'
my_app_postgresql_db:
  - db: "app"

my_app_postgresql_user:
  - name: "appuser"
    password: "randomstring"
    db: "app"
    schema: "public"
    privs: "ALL"

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

