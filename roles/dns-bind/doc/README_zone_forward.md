# DNS zone : forward

Configuration for a forward zone, which redirect all queries to one or others dns servers.  
The clients have no information about this, as the execution is managed on Bind side.

As an internal cache is maintained, it is usually used to alleviate the load of the authoritative DNS server for this zone.

## Parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_zones_dns | zones declaration | list[ object ] | [ ] |
| - | dns domain name. Ex : "sub.domain.tld" | name: "string" | mandatory |
| - | Template file. Fixed value | zone_template: "templates/etc/named/zones.d/forward-template.zone.j2" | N/A |
| - | Template zone type. Fixed value | zone_type: "zone" | N/A |
| - | The IP list of each dns server the zone must be forwarded to.<br />Ex: [ "192.168.0.1", "..." ] |  forwarders: [ "string" ] | mandatory |


## Examples

Redirect the "nowhere.com" domain to the dns hosting servers.

```
bind_zones_dns:
- name: "nowhere.com"
  zone_template: "templates/etc/named/zones.d/forward-template.zone.j2"
  zone_type: "zone"
  forwarders: [ "1.2.3.4" ]

```
