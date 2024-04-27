# DNS zone : reverse

This is the required configuration for a reverse dns zone with a config and a db template files.  
Ansible will populate their content using all the known servers from the inventory.  

Any ansible variable available to the current dns server can be reused in the zone configuration.


## Restrictions and limitations

The reverse zone is very similar to a standard zone. As such, the same limitations apply.  

Note : in case of a "ignoring out-of-zone data" error, it is usually caused by having IP addresses outside the reverse range.  
IE having an ip in "192.168.1.x" for a "0.168.192.in-addr.arpa" zone. Remove the leading "0." to cover the full 168.192 reverse range.


## Parameters

The dns domain name used must be the same for both template type.

### Template type: zone

Template file used: `templates/etc/named/zones.d/zone-template.zone.j2`

The only required information is the full reverse domain zone, in format "reverse.ip.range.in-addr.arpa"  
Ex : for 192.168.0.0/16, use 168.192.in-addr.arpa


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_zones_dns | zones declaration | list[ object ] | [ ] |
| - | Reverse ip domain name.<br />Ex : "0.168.192.in-addr.arpa" | name: "string" | mandatory |
| - | Template file. Fixed value | zone_template: "templates/etc/named/zones.d/zone-template.zone.j2" | N/A |
| - | Template zone type. Fixed value | zone_type: "zone" | N/A |
| - | Allow to transfer the zone to a list of server IPs<br />Disabled by default | allow_transfert: [ "string" ] | [ "none" ] |
| - | Allow to notify the listed server IPs for any change in the zone | allow_notify: [ "string" ] | [ "none" ] |


### Template type: db

Template file used: `templates/etc/named/zones.d/reverse-template.db.j2`

This template will create the full list of PTR record, and all the required data.  
Options are available to create an entry for each known server from the ansible's inventory.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_zones_dns | zones declaration | list[ object ] | [ ] |
| - | Reverse ip domain name.<br />The exact same value as for the zone template. | name: "string" | mandatory |
| - | Template file. Fixed value | zone_template: "templates/etc/named/zones.d/reverse-template.db.j2" | N/A |
| - | Template zone type. Fixed value | zone_type: "db" | N/A |
| - | Zone contact email address.<br />If none is defined, a fake one will be used | email: "string" | "nobody@nowhere.com" |
| - | Activate the automatic management of the zone serial, using the current time in epoch format, only when a zone change occured.<br />If used, the .serial property will be ignored. | serial_auto: boolean | yes |
| - | The version of the zone, must be updated after each change.<br />The value can be any number, as long as it is increased when a change is applied.<br />Suggestion : use the current date/time or the unix epoch time.<br />It is possible to keep the same value if the zone is not cached by other dns servers. | serial: number | mandatory without serial_auto=yes |
| - | time between refreshes for the zone, in seconds | refresh: number | 3600 |
| - | max zone duration, in seconds<br />Default is 14 days | expire: number | 1209600 |
| - | default ttl for the entries in the zone, in seconds | ttl: number | 600 |
| - | Full dns name for each ns servers managing the zone.<br />Ex: [ "dns1.corp.domain.com", "dns2.corp.domain.com" ] | ns: [ "string" ] | mandatory |
| | | | |
| - | Generate PTR for all known servers in Ansible inventory using the inventory name.<br />Set to false to deactivate. | records_inventory: boolean | true |
| - | Domain name to add to each server record, like `<server>.<domain>`<br />Otherwise the inventory name will be used as-is  | records_domain: "string" | "" |
| - | Set to false to skip ipv4 addresses from the inventory (if records_inventory=true) | allow_ipv4: boolean | true |
| - | Allowed networks for the IPv4 addresses to collect in CIDR notation.<br />Can be used to filter out external ips<br />Ex: [ "192.168.1.0/24", "192.168.5.0/24" ]<br />Default allows all ip, even docker's. | allow_ipv4_cidr: [ "string" ] | [ "0.0.0.0/0" ] |
| - | Set to false to skip ipv6 addresses from the inventory (if records_inventory=true) | allow_ipv6: boolean | true |
| - | Allowed networks for the IPv6 addresses to collect in CIDR notation.<br />Can be used to filter out external ips<br />Default allows all ip. | allow_ipv6_cidr: [ "string" ] | [ "::/0" ] |
| | | | |
| - | Allow to insert additional records, using the standard PTR syntax.<br />Can be empty. | records_raw: [ "string" ] | [ ] |


## Examples

### Complete reverse zone declaration

Using the reverse zone "0.10.in-addr.arpa"  
With the servers on the domain : "test.nowhere.corp"  
And retrieving all IPv4 addresses for known hosts in the local network.

```
bind_zones_dns:
- name: "0.10.in-addr.arpa"
  zone_template: "templates/etc/named/zones.d/zone-template.zone.j2"
  zone_type: "zone"
  # no transfer expected

- name: "0.10.in-addr.arpa"
  zone_template: "templates/etc/named/zones.d/reverse-template.db.j2"
  zone_type: "db"
  email: "admin@nowhere.corp"
  serial: 123            # single dns server, no replication, the serial has no usage
  ns:
    - "dns.test.nowhere.corp"
  # generate record automatically from the inventory
  records_inventory: yes
  records_domain: "test.nowhere.corp"
  allow_ipv4: yes
  # only the IP in the following ranges
  allow_ipv4_cidr: [ "10.0.0.0/16" ]
  # no need to bother with IPv6
  allow_ipv6: false
  # extra records - none
  records_raw: []

```

A little tip : if you want an automatic range generator for the zone name, this line will reuse the dns server own ip from the facts to generate the name.
```
- name: "{{ ansible_default_ipv4.address |ipaddr('revdns') |regex_replace('^[0-9]+\\.[0-9]+\\.', '') |regex_replace('\\.$', '') }}"
```
the .network property can also be used.


### records_raw usage

The ip address is either relative to the reverse zone, or can use the full arpa name (ending with a dot) as long as the IP is a member of the reverse zone's range.  
Ansible filters can be used as usual.

```
    records_raw:
      - "5.20                     IN    PTR      server.
      - "5.21.0.10.in-addr.arpa.  IN    PTR      server-whatever.domain.tld."
      - "{{ '10.0.22.5' |ipaddr('revdns') }}  IN    PTR      server22-5.domain.tld."

```

