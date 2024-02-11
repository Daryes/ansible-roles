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


### Mandatory parameters

None.


### Optional parameters


#### General parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_listen_ipv4_list | Listen IPv4 addresses - default to localhost IPv4 only.<br />Change it to "0.0.0.0/0" or any specific ip<br />- "0.0.0.0" without /0 will listen to localhost only<br />- "any" is accepted for all available ip<br />- "none" is accepted to disable listening for the protocol | list[ "string" ] | ["127.0.0.1"] |
| bind_listen_ipv6_list | Listen IPv6 addresses - default disable IPv6 usage.<br />Same behavior as _ipv4_list applies | list[ "string" ] | [ "none" ] |
| bind_listen_port | Listen port | numeric | 53 |
| bind_cache_max_size | Maximum memory for an individual cache database per view<br />The value can be empty (auto-managed, default to "90%"), a percent or absolute memory size (ex: 256M)<br />Requires Bind 9.16+, it must be left empty on a lower version | "string" | "" |
| bind_cache_max_ttl | Maximum TTL cache duration (in seconds)<br />Might be silently trucated by bind to 90 sec | "string" | "" |
| bind_forwarders | All purpose dns external forwarders.<br />If set to your ISP or any public ns IP, this can provide resolution for internet zones<br />For more specific settings, use a dedicated template.<br/>Example:<br />- "8.8.8.8"<br />- "9.9.9.9" | list[ "string" ] | [ ] |
| bind_zone_root_use_forwarders | Set forwarders using the default "." zone.<br />This will use forwarders for unknown zones, which is the usual configuration using your provider's dns.<br />Notice: don't forget to fill _recursion_allow_acls | boolean | yes |
| bind_options_forwarders_global | Set the forwarders globally - exclusive with _zone_root_use_forwarders<br />Warning: requests are sent first to the forwarders before being answered locally, use this setting only for a cache server. | boolean | no |
| bind_options_forwarders_global_forward_only | If _forwarders_global is activated, don't search on local zones, only use the forwarders (and cache) | boolean | yes |
| bind_options_empty_zones | Activate automatic empty zones for internal IP ranges, as per RFC 6303 and RFC 1918.<br />More information: https://kb.isc.org/docs/aa-00800 | boolean | yes |
| bind_option_empty_zones_rfc1918_force_load | force the loading of the provided rfc1912.zones file. This shouldn't be required, but some parameters combinations might prevent _options_empty_zones to work correctly | boolean | no |


#### Custom parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_raw_settings | list of custom parameters to provide to Bind. It can be anything as supported in the named.conf file at the top level.<br />Each line must be terminated with a ";" when required | list[ "string" ] | [ ] ]


Example:
```
bind_raw_settings: 
  - 'options {
  - '  dnssec-validation yes;'
  - '};'

```


#### ACL parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_options_query_allow_acls | General queries - accepted acl<br />List of acl allowed to query the server for all zones (unless if redefined in a zone) | list[ "string" ] | [ "acl_local_network" ] |
| bind_options_query_recursion_allow_acls | General query recursions - accepted acl<br />List of acl allowed to ask for recursive query to the server for all zones (unless if redefined in a zone)<br />Notice: this could be required when using forwarders | list[ "string" ] | [ "acl_local_network" ] |
| bind_option_transfert_allow_acls | General transfer zones - accepted acl<br />List of acl authorized to ask for zone transfers for all zones (unless if redefined in a zone) | list[ "string" ] | [ "none" ] |


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_acl_local_addresses | Access list for the local network usable with the name `acl_local_network` in the templates.<br />The `bind_acl_local_range_ansible_fact` variable is an IP range generated from the network facts of the first interface IP. It should allow the local network to query the dns server.<br />Examples :<br />- "192.168.0.0/24"<br />- "10.0.0.0/16" | list[ "string" ] | [ "{{ bind_acl_local_range_ansible_fact }}" ] 
| bind_acl_definitions | Additional access list definitions reusable in the templates. | list[ object ] | [ ] |
| - | acl name | name: "string" | mandatory |
| - | network IP ranges | addresses: [ "string" ] | mandatory | 

Example:
```
bind_acl_definitions:
  - name: "my_name"
    addresses:
      - "192.168.1.0/24"
      - "192.168.0.0/16"
  - name: "other acl"
    ...
```


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
* [Standard zone](README_standard_zone.md)  
  Used for a common dns zone containing a list of records names, aliases and IPs.  

* [Forward zone](README_forward_zone.md)  
  Used to forward any query to another dns server from a list.  

* [Transfer zone](README_transfer_zone.md)  
  Used to maintain a copy of an existing zone managed by another dns server.  

* [Reverse zone](README_reverse_zone.md)  
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
