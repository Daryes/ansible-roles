# Parameters

As the role is using the binary package from the system, most of the parameters are optionals, and allow Bind to work out-of-the-box.  
If fully left as default, the configuration will be highly limited.


## Mandatory parameters

None.


## Optional parameters


### General parameters

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


### Custom parameters

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

