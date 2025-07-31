# DNS zone : transfer

Configuration for a transfer zone, which copy an existing zone from another dns server to build a local cache.  
It is also known as a secondary zone, or slave zone.  

Please note that the default configuration of dns servers does not always allow the possibility to make a copy of a zone, for security reasons.  
As a requirement, this server IP must have been authorized on the servers holding the source of the requested zone.


## Parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_zones_dns | zones declaration | list[ object ] | [ ] |
| - | dns domain name. Ex : "sub.domain.tld" | name: "string" | mandatory |
| - | Template file. Fixed value | zone_template: "templates/etc/named/zones.d/transfer-template.zone.j2" | N/A |
| - | Template zone type. Fixed value | zone_type: "zone" | N/A |
| - | List of the dns server IP addresses which are the masters of the zone to copy.<br />Those servers will also be allowed to send a NOTIFY. | masters: [ "string" ] | mandatory | 
| - | Allow to force the use a specific ip from the dns server as the exit address.<br />Useful for a dns server with multiple IPs | transfer_source: "string" | "" |
| - | Same as transfer_source, for IPv6 | transfer_source_v6: "string" | "" |
| - | Minimum time in seconds before each poll to refresh the zone | refresh_min_time: "string" or number | -1 |
| - | Maximum time in seconds before a refresh.<br />Set _min_ and _max_ to `15m` to force a refresh each 15 mins | refresh_max_time: "string" or number | -1 |
| - | Allow to transfer the cached zone to the listed servers<br />Disabled by default | allow_transfert: [ "string" ] | [ "none" ] |
| - | Notify the listed server IPs for any change in the zone | also_notify: [ "string" ] | [ ] |


## Example

Create a local cache with the copy of the "nowhere.com" domain from the dns hosting servers.

```
bind_zones_dns:
- name: "nowhere.com"
  zone_template: "templates/etc/named/zones.d/transfer-template.zone.j2"
  zone_type: "zone"
  masters: [ "1.2.3.4" ]
  # depending of the version Bind only supports numbers for duration
  refresh_min_time: "1m"
  refresh_max_time: "15m"
```
