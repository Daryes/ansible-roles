# Ansible role: sys-packages-update

This role manage system updates and updating behavior.  
It is able to handle both Debian and Redhat based distributions, using apt or yum.

It is on purpose that the ansible modules related to the packages manager are not used by the main update tasks.  
Instead,  the package manager command (apt, dnf, ...)  is directly called with explicit settings.  
This is due to conflicts with system versions, update trouble with older ansible versions, lack of clarity about the updater final commands and so on.  
This said, the related ansible modules are still used for simple and maintenance tasks.  


## Requirements

The role will install its own requirements, which are system packages like python-apt and yum-utils.  

The default value of the `system_package_update_reboot_skip_hosts` parameter expects a group named `ansible` defined in the inventory, containing the ansible controllers.  
It is  used as a security to prevent rebooting the ansible server itself.  
The group can be renamed as long as the ansible servers are in this group.  

The reboot step will raise an error as a failsafe if the ansible controller is not excluded, as it cannot handle its own reboot.


## Restrictions and limitations

Kernel and other updates requiring a reboot can keep some tasks to be in the "changed" state after multiple run on modes other than 'full-with-kernel'.  
This is expected as they were not installed on those modes, but will still be present until a `full` mode is executed.


The reboot sequence will occur at the end of the playbook, after the updates are applied, and the number of server simultaneously rebooted can be limited with the parameter `system_package_update_reboot_throttle`.  
Please note this is related to the number of processes allowed to ansible, and cannot go higher. If 10 workers are allowed for Ansible, 10 servers will be rebooted simultaneously even if the reboot throttle is higher. The limitation will work when lower : with 10 workers for ansible, a throttle set to 3 will reboot only 3 servers.  
If necessary, for fine tuning, it is also possible to apply the `serial:` keyword in the playbook.  
When updating clusters, It might be more usefull to separate the cluster members in a specific group with a dedicated step in the playbook.  


Some extra commands might be required before any reboot occurs. As such, the `tasks/custom_pre_reboot.yml` task file can be updated for any desired commands.


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
| system_package_update_print_list | Print the package updates available before updating the system<br />Notice : it can generate a lot of lines in the output | boolean | no |
||
| system_package_update_task_throttle | Limit the number of simultaneous hosts for running the update tasks.<br />The value will not allow to exceed the process fork value in ansible.cfg (or command line) | numeric | 8 |
| system_package_update_reboot_throttle | Same as the task throttle, but for rebooting the hosts. | numeric | {{ system_package_update_task_throttle }} |
||
| system_package_update_ubuntu_disable_upgrade_next_lts | Ubuntu only: disable in update-manager the automatic upgrade to a new major LTS release when one is available | boolean | yes |
| system_package_update_reboot_notifier_email: Email address for the notifications on system reboot, disabled if empty.<br />For debian-type systems only and will be ignored if the 'reboot-notifier' package is missing | "string" | "" |


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
  # all valid hosts not in error must be fully updated
  any_errors_fatal: false
  serial: 1

  roles:
  - role: sys-packages-update
    vars:
      system_package_update_mode: "full-with-kernel"
      # the ansible controller is not in the clusters group, no need to skip it
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

