# Ansible role: common

This role is used to create the following groups on all servers from the inventory:

  - `os_type_<linux or windows>`
  - `os_family_<debian, rhel, ...>`
  - `os_distrib_<debian, ubuntu, centos, ...>`
  - `os_release_<version codename>`
  - `virt_<docker, kvm, vmware, ...>_<host or guest>`
  - `env_<unknown or the environment name from envir_type>`

Variables in the group_vars directory for these groups are also refreshed and applied.

It'll also fix a longtime anomaly for Windows systems, by inverting `ansible_system` and `ansible_os_family` values to be inline with their linux counterpart.
And fill the ansible_virtualization variables that were empty on Windows (might be fixed on ansible side since)


In addition, the following facts are created :  

* ansible_architecture_extra: reuse ansible_architecture value, but adjusted to the compilation architecture values, like i386, amd64, arm7, ...
  check the vars/main.yml files for the specific changes
* is_virtualmachine: set to true or false if the system is detected by ansible as a guest


At the end of this role, all servers are listed with their corresponding groups, which can be usefull instead of having to check both the inventory and the facts cache.
For example :
```
ok: [test_centos_1] => {
    "msg": "test_centos_1 system( Linux ) os_family( Redhat ) distribution( CentOS ) groups( domain, env_tests, linux, monitoring, os_distrib_centos, os_family_redhat, os_release_green-obsidian, os_type_linux, virt_docker_guest )"
}
ok: [test_debian_1] => {
    "msg": "test_debian_1 system( Linux ) os_family( Debian ) distribution( Debian ) groups( domain, env_tests, linux, monitoring, os_distrib_debian, os_family_debian, os_release_bullseye, os_type_linux, virt_docker_guest )"
}
```


## Restrictions and limitations

Ansible facts cache must be active using the following mode : gathering = smart

Unknown distributions from ansible can be handled by this role, but requires an explicit definition.
As an example, a temporary fix for Rocky Linux support is present, reusing CentOS values as Ansible doesn't support it yet.


## Requirements

None.


## Parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| envir_type | create a group based on the environment name<br />The final name will be `env_<group name>` | "string" | "unknown" |


## Usage examples

Assuming both linux and windows groups are defined in the inventory.

```
- name: All | common
  hosts: linux:windows
  remote_user: "ansible"
  become: yes
  gather_facts: true

  roles:
    - common
  tags: always


- name: Linux | my roles
  hosts: linux
  remote_user: "ansible"
  become: yes
  # facts are already in the cache, no need to retrieve them again
  gather_facts: false

  roles:
    - ...

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

