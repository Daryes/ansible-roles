# Ansible role: ntp-chrony

Install chrony on the system using the distribution packages.  
Both client (default) and server modes are supported.  
Homepage: https://chrony.tuxfamily.org/  


## Restrictions and limitations

As Chrony will take many minutes before updating the system clock, the configuration can be validated with these commands:
```
sudo chronyc activity
sudo chronyc sources -v
timedatectl
```

If systemd is present, an exclusion configuration will be added to timesyncd under the following path :
```
/etc/systemd/system/systemd-timesyncd.service.d
```
The timesyncd service will disable itself as long as the Chrony binary is present.


## Requirements

mandatory role :
* _include-pkg_install

The chrony package will be automatically installed using the distribution packages.


## Parameters


### Mandatory parameters

None.


### Optional parameters

**Server or client mode**  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| chrony_ntp_servers | Remote NTP server list with full settings<br />Ex: "pool 1.ubuntu.pool.ntp.org iburst maxsources 3"<br />Ref: https://chrony.tuxfamily.org/documentation.html  => chrony.conf | list[ "string" ] | - "server 0.pool.ntp.org"<br /> - "server 1.pool.ntp.org"<br /> - "server 2.pool.ntp.org"<br /> - "server 3.pool.ntp.org" |

If you have a local NTP server, in client mode the server list must be set only to the IP of your NTP server.


**Server mode only**  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| chrony_allow_clients_ip_ranges | IP ranges allowed to query the Chrony server.<br />Notice : if activated, localhost is always allowed.<br />Ex: [ "192.168.0.0/16", "10.0.0.0/24" ] | list[ "string" ] | [ ] |
| chrony_allow_clients_listen_ip | Local server IP to listen to queries.<br />It will only activate if `chrony_allow_clients_ip_ranges` is also used, otherwise it will restrict itself to localhost | "ip address" | "0.0.0.0" |
| chrony_allow_clients_listen_ip6 | Same as _listen_ip, for IPv6.<br />If empty, IPv6 will not be activated for Chrony | "ipv6 address" | "" |


## Usage examples

Settings for an internal network, using a "10.186.0.0/16" range.  
There will be one NTP server (10.186.0.254), connected to 4 fr servers in the ntp.org pool.  
All other servers will be connected only to the internal NTP server.  

**Server configuration**  
```
chrony_ntp_servers:
  - "server 0.fr.pool.ntp.org"
  - "server 1.fr.pool.ntp.org"
  - "server 2.fr.pool.ntp.org"
  - "server 3.fr.pool.ntp.org"


chrony_allow_clients_ip_ranges:
  - "10.186.0.0/16"
```


**Client configuration**  

```
chrony_ntp_servers:
  - "server 10.186.0.254 iburst"
```

---
## Notice: ansible and variable precedence

You can set `chrony_ntp_servers` as a global setting for all clients, then overwrite the value for the server.  
But ansible variable priority might go in the way.  

These will work :
```
# case 1 
# all has the lowest possible priority
group_vars/all => chrony_ntp_servers: [ local ntp server ]
group_vars/ntp_server => chrony_ntp_servers: [ some internet ntp server ]

# case 2 
# both groups are completely separated
group_vars/ntp_client => chrony_ntp_servers: [ local ntp server ]
group_vars/ntp_server => chrony_ntp_servers: [ some internet ntp server ]
```

This will not work
```
# case 3 
# the servers in the "linux" group are also part of the "ntp_server" group
group_vars/linux => chrony_ntp_servers: [ local ntp server ]
group_vars/ntp_server => chrony_ntp_servers: [ some internet ntp server ]

# the "linux" group value might take over the other group, with or without ansible complaining about it.
```
This has nothing to do with the role, ansible just won't allow to overwrite variables for another group at the same hierarchy level.  
While it is possible to force priorities between variables in the inventory, the results may vary.


# Licence and informations

Provided in the [meta file](meta/main.yml)

