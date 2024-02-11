# Ansible role: _include-pkg_download_install

Download and install a remote package file into the system.  
To verify if an update is required, the system package manager will be used to retrieve the already installed version, if any.  

This is an alternative to add a remote package repository, installing a specific version of a package and freezing it to prevent any update. On the other hand, this role is limited to simple packages, and will not be able to handle any requirements that could be done automatically when using a package repository.

This role is able to manage both any Debian or Redhat distribution in a single call, by expecting requested packages for both distributions (and flavors).  


To prevent usage spikes and other potential network congestion, this role also manage a binary cache.  
Any downloaded file will be stored locally on the ansible controller, then uploaded to each server when required.  
This can also be helpful when working in secure or air-gaped environments, as the cache directory can be copied from an ansible controller to another. The role will not make any internet request if the requested binary versions are present in the cache.


## Restrictions and limitations

This role is not meant to be used as-is, but to be called by another role as a function.

Insert a call in your tasks with the following command:
```
# import the package install role
- include_role: name=_include-pkg_download_install
  vars:
    <parameters>
```


A variable `_pkg_install_status` is returned, which will be set to the state "changed" if an installation occured.  
It can be used like this :
```
- shell:
    cmd: "some command to execute after the installation"
  when: _pkg_install_status is changed
```


## Requirements

None.


## Parameters

### Global parameters

These parameters should be declared one time in the inventory, under group_vars/all for example.  
Setting them in a role can lead to a discrepancy between them, using different values

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| ansible_cache_root_dir | Full path to the binary cache location | "string" | "/opt/ansible/cache" |


### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| arg_install_comment | The task name title for each task execution in the ansible output.<br />While it can be omitted, it will be visible as a "CHANGE_ME" comment | "string" |
| arg_install_pkg_name | package short name as reporter by the package manager.<br />Same name used with the package manager for a manual installation. | "string" |
| arg_install_pkg_version | Version to download and deploy, usually '1.23.4' | "string" |
| arg_install_pkg_deb_file | Debian package filename | "string" | 
| arg_install_pkg_deb_url | Url to the Debian package file.<br />While it usually contains the filename, sometimes the value can differ | "string" |
| arg_install_pkg_deb_hash | Hash for the debian file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted or left empty. | "sha256:xyz..." |
| arg_install_pkg_rpm_file | Rhel package filename | "string" | 
| arg_install_pkg_rpm_url | Url to the Rhel package file.<br />While it usually contains the filename, sometimes the value can differ | "string" |
| arg_install_pkg_rpm_hash | Hash for the Rhel file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted or left empty. | "sha256:xyz..." |
| arg_install_pkg_rpm_gpg_url | GPG key url or file for the rpm file.<br />If the key is not provided, the GPG validation will be skipped.<br />Ex : "https://packages.domain.tld/repo/gpgkey" | "string" |

Notice: All `_deb_` or `_rpm_` parameters can be omitted if there is no related host with this system.  
The role will not validate those settings.


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_install_pkg_service | Stop the designated service if an update is required.<br />The service will not be restarted by this role | "string" | "" |


## Usage examples

Installing Grafana OSS  
The variable `grafana_version` is an external parameter from the inventory, set to the requested version to install, like `"9.3.1"`

```
- include_role: name=_include-pkg_download_install
  vars:
    arg_install_comment: "Monitoring Grafana"
    arg_install_pkg_name: "grafana"
    arg_install_pkg_service: "grafana-server.service"
    arg_install_pkg_version: "{{ grafana_version }}"
    arg_install_pkg_deb_file: "grafana_{{ grafana_version }}_amd64.deb"
    arg_install_pkg_deb_url: "https://dl.grafana.com/oss/release/grafana_{{ grafana_version }}_amd64.deb"
    # arg_install_pkg_deb_hash: <= not used

    # Not using rhel/centos/rocky here
    # kept for the example, but could have been removed
    arg_install_pkg_rpm_file: "grafana-{{ grafana_version }}-1.x86_64.rpm"
    arg_install_pkg_rpm_url: "https://dl.grafana.com/oss/release/grafana-{{ grafana_version }}-1.x86_64.rpm"
    # arg_install_pkg_rpm_hash: <= not used
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

