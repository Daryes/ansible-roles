# Ansible role: sys-packages-update

This role manage system updates and updating behavior.  
It is able to handle both Debian and Redhat based distributions, using apt or yum.


## Requirements

The role will install its own requirements, which are system packages like python-apt and yum-utils.  

The default value of the `system_package_update_reboot_skip_hosts` parameter expects a group named `ansible` defined in the inventory, containing the ansible controllers.  
It can be changed to anything as long as the ansible servers are present.


## Restrictions and limitations

Kernel and other updates requiring a reboot can keep some tasks to be in the "changed" state after multiple run on modes other than 'full-with-kernel'.  
This is expected as they were not installed on those modes, but will still be present until a full mode is executed.


The reboot sequence occurs at the end of the playbook, after the updates are applied, but does not have a restriction on the number of target hosts.  
If 10 workers are allowed, 10 servers will be rebooted simultaneously.  
Having a limitation on this must be set in the playbook using `serial` (for example: `serial: 40%` ).  
If necessary, separate the members of a cluster in a specific group and have a dedicated step in the playbook.

Also, the reboot step will raise an error as a failsafe if the ansible controller is not excluded, as it cannot handle its own reboot.

Some extra commands might be required before any reboot occurs. As such, the `tasks/custom_pre_reboot.yml` task file is available.

Notice : currently, on Debian systems, the `security-only` mode has no difference with the `normal` mode, both will gave the same result as a normal update.

## Parameters

### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| system_package_update_mode | this setting will determine the update process behavior <br />Supported values :<br />`security-only` : update only packages with updates marked as security<br />`normal` : update most of installed packages, services will be restarted automatically when updated<br />`full-with-kernel` : same as normal, and also update the kernel and other packages requiring a reboot, then follow with a reboot<br /> | string | "normal" |
| system_package_update_reboot | if the kernel is updated, will follow with a reboot if required when the mode is set to `full-with-kernel` .<br />Change it to no to prevent any reboot | boolean | yes |
| system_package_update_reboot_timeout | Maximum seconds to wait for a host to reboot and respond | numeric | 600 |
| system_package_update_reboot_skip_hosts | host list to never reboot.<br />the ansible server must be present, by itself or in a group, otherwise the role will raise an error<br />syntax:<br />`  - "{{ groups['ansible'] }}"`<br />`  - "server1"`<br />`  - "{{ groups['mygroup'] }}"`<br />`  - "..."` | list[ string ] | `"{{ groups['ansible'] }}"` |
| system_package_update_ubuntu_disable_upgrade_next_lts | Ubuntu only: disable in update-manager the automatic upgrade to a new major LTS release when one is available | boolean | yes |


Regarding the Ubuntu upgrade to the next LTS, it is related to moving from 20.04 to 22.04, for example. An upgrade from 20.04.1 to 20.04.2 is a standard system update, and will happen automatically through the system's life.  
While the setting can be disabled, it doesn't support natively the switch to a different value as it is a conscious choice left to each system's administrators.  


## Usage examples

Update only packages marked as security, with a limitation at 60% to alleviate network and cpu load on hosts.  
You can also change `system_package_update_mode` value to `normal` for a standard update (without including any kernel package)
```
- hosts: linux
  become: yes  
  any_errors_fatal: true
  serial: "60%"

  roles:
  - role: sys-packages-update
    vars:
      system_package_update_mode: "security-only"
```


Full update with reboot, having a dedicated step for clusters.  
```
# all servers but not those in a cluster
- hosts: linux:!clusters
  become: yes
  any_errors_fatal: true
  serial: "30%"

  roles:
  - role: sys-packages-update
    vars:
      system_package_update_mode: "full-with-kernel"
      system_package_update_reboot_skip_hosts:
        - "my_ansible_controller"


# servers in a cluster group
- hosts: linux:&clusters
  become: yes
  any_errors_fatal: true
  serial: 1

  roles:
  - role: sys-packages-update
    vars:
      system_package_update_mode: "full-with-kernel"
      # the ansible controller is not in the clusters group, no need to skip it
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

