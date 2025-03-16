# Ansible role: _include-pkg_install

Install system packages and activating remote repositories using the usual package manager (apt/dnf/...).

This role is able to manage both any Debian or Redhat distribution in a single call, by expecting requested packages and/or external repositories for both distributions (and flavors).  

It makes also use of the installed packages caching fact, which greatly accelerate subsequent calls by comparing values instead of calling the system package manager each time.

The role will also generate a status variable, which will be set to changed if one of more packages have been installed.


## Restrictions and limitations

This role is not meant to be used as-is, but to be called by another role as a function.

Insert a call in your tasks with the following command:
```
# import the package install role
- include_role: name=_include-pkg_install
  vars:
    <parameters>


# example using the variable '_pkg_install_status' returned by the role
# using this variable is optional, and not required most of the time
- module: ...
    ... start a service or update a config file
  when: _pkg_install_status is changed
```

The role is able to register or update a custom repository or ppa in the system, but it does not have the capability to only remove them. They must be removed manually.  


When on a Debian-based distribution, if the following error "apt cache update failed" occurs (or similar), and a custom external repo/ppa is used, verify the url and version.  
This error occurs frequently when on a given repo, a specific version is requested but does not exist for the current system release, or if the repo url has changed.  


## Requirements

Same as the ansible modules `apt` and `dnf` (or the `yum` alias ) and the dedicated `*_repository` modules in the ansible.builtin collection.  
Ansible will usually install itself the requirements on the targeted hosts.


## Parameters


### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| arg_install_comment | The task name title for each task execution in the ansible output.<br />While it can be omitted, it will be visible as a "CHANGE_ME" comment | "string" |
| arg_install_packages | List of packages to install per system distribution.<br />the `common` list is applied on both Debian and Rhel systems | common: [ "string" ]<br />debian: [ "string" ]<br />redhat: [ "string" ] 


The syntax must follow this structure :
```
arg_install_packages:
  common:
    - "package1"
    - "package2"
    - "..."
  debian:
    - "package1"
    - "..."
  redhat:
    - "package1"
    - "..."
```

The structure common / debian / redhat are all mandatory.  
If there is no package to provide, use an empty list for each, like this : `common: []`


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_remove_packages | List of packages to uninstall from the system.<br />The expected structure is exactly the same as `arg_install_packages`  | common: [ "string" ]<br />debian: [ "string" ]<br />redhat: [ "string" ] | [ ] |
| arg_install_force_all | When the desired packages are already present, the repo configuration is skipped. Activate this parameter to force the execution of the full sequence.<br />When an external repo change its address abruptly, without a redirect, it will at least break the system updates. Fix the calling role with the new repo url, and use this parameter on the command like with : `ansible-playbook ... my_playbook.yml -e arg_install_force_all=yes` | boolean | no |


**Debian type repositories**  
| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_install_deb_repo | list of one or multiple Debian repo/PPA to install and activate.<br />Notice: the old name `arg_install_ppa` is still supported. | list[ object ] | [ ] |
| - | module name | name: "string" | mandatory |
| - | repo filename placed under `/etc/apt/sources.list.d/` | apt_filename: "string" | "{{ this.name }}" |
| - | repository full url.<br />Ex: `"deb http://ppa.launchpad.net/<...ppa...>/ubuntu {{ ansible_distribution_release.lower() }} main"` | apt_repository: "string" | mandatory | 
| - | url for the repo key<br />Ex: "http://domain.tld/linux/gpg" | apt_key_url: "string" | "" |
| - | dns for the key server, if used. Ex: "keyserver.ubuntu.com" | apt_key_server: "string" | "" |
| - | Key id hash | apt_key_id: "string" | "" |


**Redhat type repositories**  
| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_install_rpm_repo | list of one or multiple Rpm repo for yum/dnf to install and activate.<br />Notice: the old name `arg_install_rpm_repo` is still supported. | list[ object ] | [ ] |
| - | module name | name: "string" | mandatory |
| - | file name to store the repo information (without the path)<br />Optional when already integrated in the system repo list| yumrepo_file: "string" | "{{ this.name }}.repo" |
| - | repository full url.<br />Ex: "https://myrepo.domain.tld"<br />This parameter must be omitted for repos already integrated in the system | yumrepo_baseurl: "string" | mandatory<br />Myst be omitted if in the system |
| - | Repo description | yumrepo_description: "string" | "" |
| - | Repo url to the GPG key | yumrepo_gpgkey: "string" | "" |
| - | Activate yum/dnf hotfixes | yumrepo_module_hotfixes: boolean | False |
| - | Enable or disable a repo.<br />Mandatory for enabling a system repo | yumrepo_enabled: boolean | True |
| - | integrated module name to disable with 'dnf disable module' | yumrepo_module_disable: "string" | "" |
| - | Space separated list of packages to exclude from updates or installs.<br />Ex: `package1 packs*2"`| yumrepo_exclude: "string" | "" |



## Usage examples

**Installing nginx from the system packages**  
The package name is the same for both distributions, only Debian is requesting an additional package.
```
- include_role: name=_include-pkg_install
  vars:
    arg_install_comment: "Nginx"
    arg_install_packages:
      common:
        - "nginx"
      debian:
        - "nginx-common"
      redhat: []
```


**Installing MariaDB/MySQL from the official repo**  

```
- include_role: name=_include-pkg_install
  vars:
    arg_install_comment: "MySQL/MariaDB"
    arg_install_packages:
      common:
        # install some requirements
        - "rsync"
        - "curl"
        - "socat"
      debian:
        - "software-properties-common"
        - "mariadb-server-10.10"
        - "mariadb-client-10.10"
        - "mariadb-server"
        - "mariadb-client"
        - "mariadb-backup"
      redhat:
        # no packages using the version explicitly
        - "MariaDB-server"
        - "MariaDB-client"
        - "MariaDB-backup"
    #
    arg_install_deb_repo:
      - name: mariadb-org
        apt_key_url: "https://mariadb.org/mariadb_release_signing_key.asc"
        apt_repository: "deb https://deb.mariadb.org/10.10/debian {{ ansible_distribution_release |lower }} main"
        apt_filename: "mariadb.list"
    #
    arg_install_rpm_repo:
      - name: mariadb-org
        yumrepo_file: "mariadb-org"
        yumrepo_description: "MariaDB from mariadb.org"        
        yumrepo_baseurl: "https://yum.mariadb.org/10.10/rhel/$releasever/$basearch"
        yumrepo_gpgkey: "https://yum.mariadb.org/RPM-GPG-KEY-MariaDB"
        yumrepo_module_hotfixes: true
```

Given the number of parameters, it is possible to move each settings to dedicated variables under `vars/`  
The task call can be reduced to something like this :
```
- include_role: name=_include-pkg_install
  vars:
    arg_install_comment: "MySQL/MariaDB"
    arg_install_packages: "{{ role_mysql_install_packages }}"
    arg_install_deb_repo: "{{ role_mysql_install_ppa }}"
    arg_install_rpm_repo: "{{ role_mysql_install_yumrepo }}"
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

