# Ansible role: _include-compose_deploy

Install a docker-compose.yml template and its configuration to a given directory.
Then update the images, and start the compose services.

The general status will be return in the following fact: `_compose_deploy_status`  
It will be set to the `changed` state if the files were updated.  
Updating the images or starting the containers will not be reported as a change.


## Restrictions and limitations

This role is not meant to be used as-is, but to be called by another role as a function.

Insert a call in your tasks with the following command:
```
# import the package install role
- include_role: name=_include-compose_deploy
  vars:
    <parameters>
```


## Requirements

Docker and docker-compose must be already installed on the target hosts.


---
## Parameters


### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| arg_compose_comment | The task name title for each task execution in the ansible output.<br />While it can be omitted, it will be visible as a "CHANGE_ME" comment | "string" |
| arg_compose_deploy_dir | Installation directory - usually "/opt/myapp".<br />The path will be created if required | "string" |
| arg_compose_main_template | Path to the "docker-compose.yml" template to use.<br />A static, non-templated file can be used.<br />The filename will be reused, removing any ".j2" suffix | "string" |


### Optional parameters


**General parameters**  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_compose_env_template | Path to the ".env" template file. The name will be reused.<br />The name can differ from ".env", an extra ".env" symlink will be created. | "string" | "" |
| arg_compose_conf_sensitive | Set to yes to have both the _env_template and _conf_extra set as sensitive, this will limit the ansible log output | boolean | no |
| arg_compose_force_pull_always | force an image pull on each run, instead of only when one of the configuration file is updated.<br />Notice: to have this feature working as intented, the image tag must also be set to a generic tag, like "latest" or "10" instead of "10.5.1" | boolean | no |
| arg_compose_force_pull_on_change | Same as _force_pull_always, but only when a configuration change occurs, and it will update all the images.<br />If set to no, the image will be pulled only when docker itself decides to do it. | boolean | yes |
| arg_compose_force_stop_on_change | The standard behavior is to only execute a "compose up -d".<br />Set this to "yes" to have a "compose down" first when configuration files from  _env_template and _conf_extra are updated | boolean | no |
| arg_compose_up_execute | set to "no" to skip the "docker-compose up -d" execution.<br />If skipped, the calling role will have to start the container itself | boolean | yes |
| arg_compose_up_options | extra options for the "docker-compose up -d" command.<br />Anything listed in "docker-compose up --help" goes here | "string" | "" |


**docker-compose extra configuration**  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_compose_conf_extra | Settings for installing extra configuration files | list[ object ] | [ ] |
| - | template source to use |  template: "string" | mandatory |
| - | destination directory |  dest: "string" | mandatory |
| - | file owner | owner: "string" | "root" |
| - | file group | group: "string" | "{{ owner }}" |
| - | file access mode (in octal) | mode: "string" | "0644" |

The destination is a subdirectory, not the full path. It will be placed under the path in "arg_compose_dir".

Syntax examples:
```
arg_compose_conf_extra:
  - { template: "templates/path/to/template.file", dest: "subdir/file.conf" }
  - { template: "templates/path/to/template2.j2",  dest: "subdir/file2.ini", owner: "myuser", group: "mygroup", mode: "0640" }
```

## Usage examples

Deployement of the [Yopass](https://github.com/jhaals/yopass) application
```
- include_role: name=_include-compose_deploy
  vars:
    arg_compose_comment: "Yopass"
    arg_compose_deploy_dir: "/opt/yopass"
    arg_compose_main_template: "templates/docker-compose.yml.j2"
    arg_compose_env_template: "templates/yopass.conf.j2"
    # no pull and no stop on any change, let the "up"  pull the image when it will be required
    arg_compose_force_stop_on_change: no
    arg_compose_force_pull_on_change: no


# extra actions if the deploy role applied some changes
- task:
    (...)
  when: _compose_deploy_status is changed

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

