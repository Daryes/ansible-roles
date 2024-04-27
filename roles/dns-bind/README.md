# Ansible role: dns-bind

Install the dns server Bind with a full configuration, and generate dns zones using the ansible inventory.


## Restrictions and limitations

As the role make use of the distribution packages, the configuration is located under `/etc/bind` for Debian, and `/etc/named` for RHEL/Centos.  
But to alleviate differences between distributions, the configuration itself is fully contained in the corresponding directory, using its own structure.  
Multiples changes are applied, and some files from the packages are ignored. For example, the zones are located under `/etc/????/zones.d`, and the logs are under `/var/log/named/`  
As such, is it recommended to use the role itself to manage the content.  


## Requirements

System package on the Ansible controller:
* python3-netaddr  
  Installed either with pip or apt/yum

Ansible collections :
* ansible.netcommon

Mandatory role :
* _include-pkg_install


## Parameters

Please refer to the following dedicated pages :
* [**Mandatory and optional parameters**](doc/README_general.md)  
  General parameters for the role

* [**ACL parameters**](doc/README_acl.md)  
  Specific ACL parameters


## DNS zone parameters

All zones are declared as a set of parameters in a list, using of one of the provided templates, or an external one.  
Each template will generate a file under /etc/bind/zones.d/ using the "name" property as the filename.  

Files for unused zones will not be removed, without causing any interference :  
the main configuration is always updated to load only the active zone files.  


### Zone declarations

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_zones_dns | list of dns zones to create.<br />The properties "name", "file" and "type" are mandatory, all other properties are specific to each template file | list[ object ] | [ ] |
| - | Domain name of the zone.<br />If a zone uses multiple templates, the name must be reused. | name: "string" | mandatory |
| - | Template file. | zone_template: "string" | mandatory |
| - | Template type.<br />See the dedicated section for the current supported types. | zone_type: "string" | mandatory |

examples:
```
bind_zones_dns:
  - name: "mysub.domain.tld"
    zone_template: "templates/etc/named/zones.d/sample.domain.zone.j2"
    zone_type: "zone"
    ... other template settings

  - name: "mysub.domain.tld"
    zone_template: "templates/etc/named/zones.d/sample.domain.db.j2"
    zone_type: "db"
    ... other template settings

  - name: "another.doma.in"
    zone_template: ...
```


## Zones types and templates

There are currently 3 types of supported zones, each with different settings.
Each make use of a specific template type, or uses multiple of them

The following zones can be created :
* [**Standard zone**](doc/README_zone_standard.md)  
  Used for a common dns zone containing a list of records names, aliases and IPs.  

* [**Forward zone**](doc/README_zone_forward.md)  
  Used to forward any query to another dns server from a list.  

* [**Transfer zone**](doc/README_zone_transfer.md)  
  Used to maintain a copy of an existing zone managed by another dns server.  

* [**Reverse zone**](doc/README_zone_reverse.md)  
  Used for a reverse dns zone containing a list of PTR records.


Please refer to the dedicated documentation files for each zone.

In addition, the following pre-configured db files are available for custom zones  
under `/etc/bind/zones.d` ( or `/etc/named/zones.d` on rhel/centos) :  
* default.zone-empty.db
* default.reverse-empty.db
* default.zone-localhost.db
* default.reverse-127.db


## Usage examples

The following is limited to the general settings, for a local network usage only.  
Other settings are related to the zone type, which should be a standard zone for this usage.

```
bind_listen_ipv4_list: [ "0.0.0.0/0" ]
bind_listen_ipv6_list: [ "none" ]
bind_cache_max_size: "256M"
bind_forwarders: [ "192.168.0.1" ]
bind_acl_local_addresses: [ "192.168.0.0/16" ]

bind_zones_dns:
  - ... standard zone configuration,
    ... as per the related documentation
```


# Licence and informations

Provided in the [meta file](meta/main.yml)
