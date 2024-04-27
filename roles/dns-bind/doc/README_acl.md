# Parameters

## Optional parameters

### ACL parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_acl_local_addresses | Access list to populate the local network acl named `acl_local_network`<br />The `bind_acl_local_range_ansible_fact` variable is an IP range generated using the network facts of the first interface IP. It should allow the local network to query the dns server.<br />Examples :<br />- "192.168.0.0/24"<br />- "10.0.0.0/16" | list[ "string" ] | [ "{{ bind_acl_local_range_ansible_fact }}" ] 
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


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| bind_options_query_allow_acls | General queries - accepted acl<br />List of acl allowed to query the server for all zones (unless if redefined in a zone)<br />the acl named `acl_local_network` is available globally and is populated by `bind_acl_local_addresses` | list[ "string" ] | [ "acl_local_network" ] |
| bind_options_query_recursion_allow_acls | General query recursions - accepted acl<br />List of acl allowed to ask for recursive query to the server for all zones (unless if redefined in a zone)<br />Notice: this could be required when using forwarders | list[ "string" ] | [ "acl_local_network" ] |
| bind_option_transfert_allow_acls | General transfer zones - accepted acl<br />List of acl authorized to ask for zone transfers for all zones (unless if redefined in a zone) | list[ "string" ] | [ "none" ] |

