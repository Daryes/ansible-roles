# DNS zone : standard

This is the required configuration for an usual dns zone with a config and a db template files.  
Ansible will populate their content using all the known servers from the inventory.  

Any ansible variable available to the current dns server can be reused in the zone configuration.


This template also has the capability to search in the whole inventory, looking for a particular variable name based on group membership's, following this format: `<a_group_name>_bind_zones_extra`.  
As such, it is possible to declare additional dns records in multiple locations, either globally, or only for a specific group or host.  

Example: a host member of the inventory groups "group1" and "group2" will be scanned for variables named :
* group1_bind_zones_extra
* group2_bind_zones_extra

The additional informations from those declaration will be assembled with the related DNS zones declared in `bind_zones_dns`  
Also, if a variable `group9_bind_zones_extra` exists for this host, as the host is not a member of group9_test, the variable will be ignored.  


## Restrictions and limitations

To be able to generate a record for the servers in the inventory, some configuration changes are required in ansible.cfg:
* in the playbook, one of the play must have the "gather_facts: yes" setting
* in ansible.cfg: "gathering  = smart" and "fact_caching = jsonfile" (or anything different than "memory")  

The .cfg changes are not required if a "gather_facts" step targeting all the hosts is present in the same playbook.  
Otherwise, some hosts might be missing from the dns zone.

While Windows hosts are supported, results may varies. This is due to the Ansible limited support of Windows servers.


## Parameters

The dns domain name used must be the same for both template type.

### Template type: zone

Template file used: `templates/etc/named/zones.d/zone-template.zone.j2`

The only required information is the full domain name.  


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_zones_dns | zones declaration | list[ object ] | [ ] |
| - | dns domain name. Ex : "sub.domain.tld" | name: "string" | mandatory |
| - | Template file. Fixed value | zone_template: "templates/etc/named/zones.d/zone-template.zone.j2" | N/A |
| - | Template zone type. Fixed value | zone_type: "zone" | N/A |
| - | Allow a list of host IPs to submit dynamic updates to this server<br />Disabled by default | allow_update: [ "string" ] | [ "none" ] |
| - | Allow a list of server IPs to read and transfer the zone<br />Disabled by default | allow_transfert: [ "string" ] | [ "none" ] |
| - | Notify the listed server IPs for any change in the zone | also_notify: [ "string" ] | [ ] |


### Template type: db

Template file used: `templates/etc/named/zones.d/zone-template.db.j2`

This template will create the full list of dns record, and all the required data.  
Options are available to create an entry for each known server from the ansible's inventory.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_zones_dns | zones declaration | list[ object ] | [ ] |
| - | dns domain name. Ex : "sub.domain.tld"<br />Exact same value as for the zone template. | name: "string" | mandatory |
| - | Template file. Fixed value | zone_template: "templates/etc/named/zones.d/zone-template.db.j2" | N/A |
| - | Template zone type. Fixed value | zone_type: "db" | N/A |
| - | Zone contact email address.<br />If none is defined, a fake one will be used | email: "string" | "nobody@nowhere.com" |
| - | Activate the automatic management of the zone serial, using the current time in epoch format, only when a zone change occured.<br />If used, the .serial property will be ignored. | serial_auto: boolean | yes |
| - | The version of the zone, must be updated after each change.<br />The value can be any number, as long as it is increased when a change is applied.<br />Suggestion : use the current date/time or the unix epoch time.<br />It is possible to keep the same value if the zone is not cached by other dns servers. | serial: number | mandatory without serial_auto=yes |
| - | time between refreshes for the zone, in seconds | refresh: number | 3600 |
| - | max zone duration, in seconds<br />Default is 14 days | expire: number | 1209600 |
| - | default ttl for the entries in the zone, in seconds | ttl: number | 600 |
| - | Full dns name for each ns servers managing the zone.<br />Ex: [ "dns1.corp.domain.com", "dns2.corp.domain.com" ] | ns: [ "string" ] | mandatory |
| - | Full dns name of the mail exchangers, with the priority in front if required. Can be empty.<br />Ex : [ "10 mail1.corp.domain.com", "30 mail2.corp.domain.com" ] | mx: [ "string" ] | [ ] |
| - | spf records, taken as is. can be empty | spf: [ "string" ] | [ ] |
| | | | |
| - | Generate records for all known servers in Ansible inventory<br />Set to false to deactivate. | records_inventory: boolean | true |
| - | Set to false to skip ipv4 addresses from the inventory (if records_inventory=true) | allow_ipv4: boolean | true |
| - | Allowed networks for the IPv4 addresses to collect in CIDR notation.<br />Can be used to filter out external ips<br />Ex: [ "192.168.1.0/24", "192.168.5.0/24" ]  | allow_ipv4_cidr: [ "string" ] | [ "0.0.0.0/0" ] |
| - | Generate an extra `*.<server_name>` record for known servers with IPv4 address  | allow_ipv4_wildcard: boolean | false |
| - | Set to false to skip ipv6 addresses from the inventory (if records_inventory=true) | allow_ipv6: boolean | true |
| - | Allowed networks for the IPv6 addresses to collect in CIDR notation.<br />Can be used to filter out external ips | allow_ipv6_cidr: [ "string" ] | [ "::/0" ] |
| - | Generate an extra `*.<server_name>` record for known servers with IPv6 address  | allow_ipv6_wildcard: boolean | false |
| | | | |
| - | Add additional records, using the standard DNS syntax.<br />Can be empty. | records_raw: [ "string" ] | [ ] |


**Inventory additional records**  

This allow to create partial definitions in groups, without having to change the main configuration.  
The role will search for the definitions and add them, after removing the duplicates.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<group_name>_bind_zones_extra` | list of zone with additional raw dns records | list[ object ] | [ ] |
| - | dns domain name. Ex : "sub.domain.tld"<br />Exact same value as for the zone template. | name: "string" | mandatory |
| - | Template zone type. Fixed value | zone_type: "db" | N/A |
| - | Add additional records, using the standard DNS syntax.<br />Can be empty.<br />Duplicates will be removed. | records_raw: [ "string" ] | [ ] |

Any valid value can be used in records_raw, even comments (using %).  
Given they will complete the main zone, it is possible to use only CNAME records here.  
Duplicates will be removed when combining all `records_raw` but only for those parsed from the whole inventory, the main definition will not be altered.  

Please note identicals records of type A are accepted, but not for other type. CNAME duplicates, for example will generate an error.


## Examples

### Complete dns zone declaration

Using the dns name : "test.nowhere.corp"  
And retrieving all IPv4 for known hosts in the local network.

```
bind_zones_dns:
- name: "test.nowhere.corp"
  zone_template: "templates/etc/named/zones.d/zone-template.zone.j2"
  zone_type: "zone"
  # no transfer allowed

- name: "test.nowhere.corp"
  zone_template: "templates/etc/named/zones.d/zone-template.db.j2"
  zone_type: "db"
  email: "admin@nowhere.corp"
  serial: 123            # single dns server, no replication, the serial has no usage
  ns:
    - "dns.test.nowhere.corp"
  # no mx nor spf usage
  # generate record automatically from the inventory
  records_inventory: yes
  allow_ipv4: yes
  # only the IP in the following ranges
  allow_ipv4_cidr: [ "10.11.1.0/24", "10.11.2.0/24" ]
  # no need to bother with IPv6
  allow_ipv6: false
  # extra records
  records_raw: 
    - "ansible   IN CNAME srv-ansible"
    - "dns       IN CNAME srv-bind"
```


### records_raw usage

As declarations in the ansible inventory allow the use of filters, it is possible to have simple entries to more complex ones.

```
    (...)
    records_raw:
      - "test           IN    CNAME    server1"
      - "10.11.12.13    IN    A        static-server"
      - ...

      # get the servers from the 'test' group, remove a 'prefix-' string and generate a CNAME entry: server-name  IN CNAME prefix-server-name
      - "{{ groups['test'] |unique |select('search', 'prefix-') |map('regex_replace', '^([a-z]+-)(.*)$', '\\g<2>     IN  CNAME   \\g<1>\\g<2>') |list }}"
```

### partial records in inventory

Given the current host is a member of `my-group` and the main configuration manage a "zone.domain.tld" zone.  
Add these lines to the group_vars/my-group file in the inventory :

```
my-group_bind_zones_extra:
  - name: "zone.domain.tld"
    zone_type: "db"
    records_raw:
      - "test2          IN    CNAME    server2"
      - "10.11.12.134   IN    A        static-server"
      - ...
```

The raw records will be inserted at the end of the configuration for the zone.  
In case the entries are missing in the db file, make sure the zone name is the exact same, and the `zone_type: "db"` is also present.  

Please remember the usual DNS rules still apply. Especialy as, while duplicates are removed from the combinated result, the main configuration will also generate its own records.

