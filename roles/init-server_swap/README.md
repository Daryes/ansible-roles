# Ansible role: init-server_swap

This role manage swap files.  
It will create a swap file if no swap is used, or generate an additional swap file if the current swap size is lower than requested.  
All created swap files will be registered in the fstab, allowing to be used for the next reboot.

An automatic numbering will be applied to manage multiple files after the first one.  
For example, creating a 3rd swap file will use the name "/swapfile.3"


Please note no change will occur if the requested size is lower than the current swap amount, 
as the role does not manage any swap reduction.  
If required, it is possible to disable the swap, then recreate a new one, with a lower size.


## Requirements

The "sed" command-line utility is required on the targeted hosts.


## Restrictions and limitations

The role will check if enough disk space is available on the targeted filesystem.
As such, the swapfile must be located at the root of the targeted filesystem.
Using a subdirectory will raise an error for not enough space available.

The role will not check if the generated file number already exists due to a manual intervention.

It is also not able to change existing swap file locations (new one can still be created elsewhere).
For such case, either the _file parameter must be changed before creating the first swap file,
or use the swap disabling option (with great care as it can lead to a out of memory situation, then a system crash).

To alleviate interferences due to rounding numbers, a swap increase will be applied only if the difference is more than 10mb.
Example: an existing swap with a total size of 1020mb will be skipped if 1024mb is requested.


## Parameters

### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| system_swap_file | location and name for the swap file | string | "/swapfile" |
| system_swap_size_mb | swap size (value is in mb) <br />If set to 0, the creation will be skipped | number | 0 |
| system_swap_disable_force | set to "yes" and also "system_swap_size_mb=0" to disable all swap | boolean | no |


## Usage examples

Set the swap to 2048 Mb at the default location (which is "/swapfile" ).
```
system_swap_size_mb: 2048
```

Using the system memory as a starting point, allocating half of it, on a dedicated partition (already mounted).  
While expecting a number, the role alleviate the possibilty the size might be presented as a string.
```
system_swap_size_mb: "{{ ansible_memory_mb.real.total // 2 }}"
system_swap_file: "/myfs_mount/swapfile"
```

Fully disable the swap. All swap files will also be removed and unregistered from the fstab.
```
system_swap_file: 0
system_swap_disable_force: yes
```


---
# Licence and informations

Provided in the [meta file](meta/main.yml)

