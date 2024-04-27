# Ansible role: init-server_drive_format

This role will initialize an empty drive and create a lvm partition formated with the requested filesystem using the full space.  
The /etc/fstab file will be updated accordingly.  

Multiples checks are done at the beginning: if any partition or block device exists, even if some free space remain, or anything suspect is detected, no action will be applied on the target drive.  
If a drive is not present, all steps will be skipped for this drive without raising any error.  

The fstab will be updated using the LVM informations, only if the vg/lv defined exists.  
Otherwise it will also be skipped without raising an error.

The role is also able to search for a drive by its own id from `/dev/disk/by-id` instead of its device name,  
which can be useful with some hardware cards or virtualization plateform having the unusual behavior to flip the device name between the disks.


Notice: when using more than one data drive, the following error might occurs on the second data drive:  
`   Physical volume "/dev/sd?" still in use (...) "msg": "Unable to reduce <vg name> on /dev/sd<previous drive>.`  
This is caused by having the same VG name for both drives. Each drive must have a unique VG name.


## Restrictions and limitations

Using lvm is currently a mandatory requirement.

The role does not handle having lvm VG expanded on multiple drives. The layout is: `1 drive = 1 dedicated VG = 1 dedicated LV`

Due to some lvm quirks, all existing VG are activated by this role.

Other restrictions are listed in the "LVM quirks" section.


## Requirements

The following system packages must already be installed :  
* lvm2
* parted
* fdisk
* util-linux (for blkid)

Same for any driver related to any non-native partition types which would be requested (like btrfs).  


## Parameters


### Mandatory parameters

None.  
With no parameters, the role will skip the target server.


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| system_drives_initialize | list of drive devices to initialize | list[ object ] | [ ] |
| - | device name located under /dev/<br/>Used only when formatting the drive.<br />Example: "sdb" | device_name: "string" | mandatory |
| - | Full device physical ID, as listed under `/dev/disk/by-id/`<br />If a drive has multiples ID, only one of them is enough.<br />Used for some virtualization solutions where the device name can change after a reboot.<br />Ex: "scsi-SQEMU_QEMU_HARDDISK_af1...ec2"  | device_id: "string" | "" |
| - | Mount point on the filesystem, will be created if absent. | mount_point: "string" | mandatory |
| - | Fstab mount options, as supported by mount.<br />If empty or omitted, the default will be used. | mount_options: "string" | "defaults,relatime" |
| - | Filesystem name. This parameter is mandatory.<br />Example: "ext4" or "xfs" or any other type supported by the system | fstype: "string" | mandatory |
| - | Optional tuning options for the new fs, passed as-is to mkfs.<br />Currently default to these values, otherwise empty :<br />- for ext4: `-O 64bit`<br />- for xfs: `-b 4096` | fstype_options: "string" | special or "" |
| - | LVM vg name, must be unique on the host and follow the documentation in lvm(8)/valid names  | lvm_vgname: "string" | mandatory |
| - | LVM lv name, same as lvm_vgname | lvm_lvname: "string" | mandatory |

Regarding 

## Usage examples

Being already initialized, the system drive should not be present.  
Any other drive can be listed.

```
system_drives_initialize:
  - device_name: "sdb"
    mount_point: "/data"
    fstype: "xfs"
    lvm_vgname: "vgdata"
    lvm_lvname: "lvdata"

  - device_name: "sdc"
    mount_point: "/opt"
    mount_options: "defaults,noatime"
    fstype: "ext4"
    fstype_options: "-O 64bit"   # ensure the ext4 partition is able to grow over 16 Tb
    lvm_vgname: "vgstorage"
    lvm_lvname: "lvopt"

  - device_name: "sdd"
    device_id: "scsi-SQEMU_QEMU_HARDDISK_af10e434-431b-48aa-bc8a-980684fe4ec2"
    mount_point: "/mnt/storage"
    mount_options: "defaults,noatime"
    fstype: "ext4"
    fstype_options: "-O 64bit"
    lvm_vgname: "vg{{ inventory_hostname |regex_replace('[_-]|\..*','') }}"
    lvm_lvname: "storage"

  - device_name: "sdx"
    device_id: "KVM_af10e434-431b-48aa-b"
    ...
```


## LVM quirks

### VG removal and duplicate names

LVM does not play nice when volumes are frequently attached and removed.  
Even less when the volume groups reuse the same name, for different purposes.  

To reduce any possible interference, each volume group should use an unique name.  
Using a value similar to this example would help :  
```
    lvm_vgname: "vg{{ inventory_hostname |regex_replace('[_-]|\..*','') }}"
```
With this, the server name in the ansible inventory is reused after being cleaned from special chars, ensuring a unique name on each server.  


### Dedicated VG

When adding multiple LV in the same VG, LVM expects an edition of the current VG.  
This role currently cannot handle this requirement, thus each VG declaration for the same computer must be unique.  
Using the same example as before, the VG name should be suffixed with the purpose of the drive (data, log, ...)  
This would give a final value similar to :
```
    lvm_vgname: "vg{{ inventory_hostname |regex_replace('[_-]|\..*','') }}data"
```

### Drive letters and lvm volumes

In some particular circumstances, there can be a drift with the drive letters (sdb, sdc, ...) for drives presented as removable devices.  
This is caused by the Linux kernel, sometimes due to incomplete drivers.  
As such, the role is able to make use of the device physical ID, allowing to select the correct device.  
After the drive has been initialized, mounting the volume and updating the /etc/fstab file will rely only on the lvm names, which are immune to any drive letter change.

### Valid vg and lv names

In addition of the reserved characters and words, as described in the lvm(8) manual, some characters in the vg/lv names can cause lvm to apply a slighly altered name.  
The role only reuse the provided parameters, and will not retrieve the final lvm names.  
As such, it is recommended to only use letters and numbers.


# Licence and informations

Provided in the [meta file](meta/main.yml)

