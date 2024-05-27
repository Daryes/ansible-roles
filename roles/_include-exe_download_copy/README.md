# Ansible role: _include-exe_download_copy


Download and install a binary executable file to a specific location.  
To verify if an update is required, the installed executable, if present, will be run with a --version parameter


To prevent usage spikes and other potential network congestion, this role also manage a binary cache.  
Any downloaded file will be stored locally on the ansible controller, then uploaded to each server when required.  

This can also be helpful when working in secure or air-gaped environments, as the cache directory can be copied from an ansible controller to another. The role will not make any internet request if the requested binary versions are present in the cache.

The role also has a control step for checking the presence or validity of some parameters.


## Restrictions and limitations

This role is not meant to be used as-is, but to be called by another role as a function.

Insert a call in your tasks with the following command:
```
# import the package install role
- include_role: name=_include-exe_download_copy
  vars:
    <parameters>
```


Notice when downloading compressed archives:  
The parameter `arg_install_exe_compressed_archive_skip_subdirs_arg` has a preset for a tar (or tar.gz) archive to remove the first directory level.  
The parameter must be changed if the archive is not a tar/tar.gz file, like a .zip. An error will occur otherwise.  
If not used, emptying it with `''` might generate an error with some archivers, like unzip.  
In such case, use a non-intrusive parameter like verbose or quiet. For unzip, use `'-q'`


## Requirements

Any archive format requirements must already be installed on the target hosts, like tar, unzip, 7za, ...  
They are not required on the ansible server, as the files will only be copied as-is.


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
| arg_install_exe_fullpath | Full path of the location the binary file will be installed to. | "string" |
| arg_install_exe_version | Version to download and deploy, usually '1.23.4' | "string" |
| arg_install_exe_download_url | Remote url to download the file if missing from the cache | "string" | 


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| arg_install_exe_version_same_for_all | It is expected the version to install is the same for all hosts.<br />Set this to false if it not the case to prevent an error with a missing file in the cache | boolean | true |
| arg_install_exe_hash | Hash for the downloaded file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| arg_install_exe_owner | Owner of the destination directory.<br />The user must already exists.<br />Default to the ansible user | "string" | "{{ ansible_user_id }}" |
| arg_install_exe_group | Group of the destination directory.<br />The group must already exists | "string" | "{{ arg_install_exe_owner }}" |
| arg_install_exe_mode| Access mode (octal) of the destination directory. | "string" | "0755" |
| arg_install_exe_version_arg | Command parameter to show the version information | "string" | "--version" |
| arg_install_exe_version_string | Version string number to compare with the installed binary.<br />Notice: the value will be used as a regex.  | "string" | "{{ arg_install_exe_version }}" |
| arg_install_exe_version_altcmd | Alternate command to verify the version, default to the exe file itself.<br />See the example after for usage with archives like java jar | "string" | "{{ arg_install_exe_fullpath  }}" |
| arg_install_exe_is_compressed_archive | Set to 'yes' if the downloaded file is a compressed archive" | boolean | no |
| arg_install_exe_compressed_archive_binaries_to_link | list of file in the archive to be symlinked under the installation directory.<br />Example :  [ "binary1", "bin/file2", ... ] | list[ "string" ] | [ ] |
| arg_install_exe_compressed_archive_skip_subdirs_arg | Specify the archiver parameter to skip 1 or more directory levels<br />The default value is for GNU tar.<br />Set to empty `""` if unwanted | "string" | "--strip-components=1" |
| arg_install_exe_symlink_to_usr_bin | Set to 'yes' to create an additional symlink under /usr/bin | boolean | no |
| arg_install_exe_service | Stop the designated service if a file update is required.<br />The service will not be restarted by this role | "string" | "" |


For skipping subdirectories, GNU tar uses `--strip-components=1` to remove the first directory level.  
With zip/unzip, only the parameter `-j` exists, and will remove any directory information and unpack all the files on a single level.

## Usage examples

**Installing docker-compose v2**  
The variable `docker_compose_version` is an external parameter, which is set to "2.6.1"  
Same for the variable `docker_compose_version_hash`, which is set to "sha256:ed79398562f3a80a5d8c068fde14b0b12101e80b494aabb2b3533eaa10599e0f"

```
- include_role: name=_include-exe_download_copy
  vars:
    arg_install_comment: "Docker compose"
    arg_install_exe_fullpath: "/usr/local/bin/docker-compose"
    arg_install_exe_version: "{{ docker_compose_version }}"
    arg_install_exe_hash: "{{ docker_compose_version_hash }}"
    arg_install_exe_version_arg: "--version"
    arg_install_exe_download_url: "https://github.com/docker/compose/releases/download/v{{ docker_compose_version }}/docker-compose-linux-x86_64"
    # alleviate systems which do not have /usr/local/bin in the global PATH
    arg_install_exe_symlink_to_usr_bin: yes
```


**Installing prometheus node_exporter**  
It comes in a tar.gz archive, containing multiples files.  
Only the main binary is required, others are either informational, or a configuration sample.

For this example, no variable is used.
```
- include_role: name=_include-exe_download_copy
  vars:
    arg_install_comment: "Monitoring Prometheus agent | node"
    arg_install_exe_fullpath: "/usr/local/bin/node_exporter"
    arg_install_exe_version: "1.5.0"
    # not caring about the hash
    arg_install_exe_version_arg: "--version"
    arg_install_exe_download_url: "https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz"
    arg_install_exe_is_compressed_archive: yes
    arg_install_exe_compressed_archive_binaries_to_link: [ "node_exporter" ]
    arg_install_exe_service: "node-exporter.service"
```
The result will be the archive extracted under `/usr/local/bin/node_exporter-archive`, and the node_exporter binary linked as `/usr/local/bin/node_exporter`  
The node-exporter service will also be stopped if an update was required.


**Handling Java jar and other package files**  
Some files might not allow to be executed with a version parameter, like jar files.  
As long they can be manipulated with a shell command to extract the version number, such files are supported by this role.  

Example for a standard jar file having the version contained in the manifest, which is a text file in the jar.  
Use the following :  

```
- include_role: name=_include-exe_download_copy
  vars:
    ...
    arg_install_exe_version_altcmd: "unzip -p /full/path/to/my.jar META-INF/MANIFEST.MF"
    arg_install_exe_version_arg: "|grep -i 'Implementation-Version' "
```

This will unzip to the stdout the manifest, then piped to grep for filtering to the line with the version string.  
The role will then look for the version number in the filtered output.


# Licence and informations

Provided in the [meta file](meta/main.yml)

