# Ansible role: web-nginx


### Specific parameters: stream proxy

Related templates:  
* **web-nginx/templates/etc/nginx/sites-available/stream-servers.conf.j2**

The role will use this template to create an entry for a 1:1 port mapping between the reverse proxy side and the target server side, working in stream (binary) mode.  
If a range of multiple ports is specified, the template will create as much entries as needed.

Notice: while collecting all declarations in the same manner as the reverse proxy virtual host, the settings are managed by a unique template on the nginx server. As such, any duplicate declaration will be kept, and might be refused by nginx.  


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<group_name>_stream_nginx` | list of definition to bind ports as a stream proxy | list: [ object ] | [ ] |
| - | a name without spaces - must be unique<br />Example: "some_unique_name" | name: "string" | mandatory |
| - | port list to bind.<br />Example: [ 9080, 9043, 8888 ] | ports: [ "string" or number ] | mandatory |
| - | target ip or dns for the stream proxy pass directive |  proxy_pass_ip: "string" | mandatory |
| - | the nginx server IP the stream proxy will bind to | listen_ip: "string" | "*" |
| - | timeout to connect to the target | proxy_connect_timeout: "string" | "1s" |
| - | network timeout for the target to answer | proxy_timeout: "string" | "3s" |
| - | remap the nginx backend port to connect to a different port on the target<br />Example:<br />proxy_backend_ports: [ "port_9080": 8080, "port_8888": 15088 ] | proxy_backend_ports: [ "port_XXXX": number ] | [ ]
| - | Raw nginx parameters, taken as-is.<br />Each line must be terminated with a ";" | raw_settings: [ "string" ] | [ ] |


The parameter `backend_ports` can be used only for the ports requiring a remapping.  
While it is provided for special cases, the main usage will be to have the same port for both listening and the target.


---
## Usage examples

The examples are valid for any host member of the "mygroup" inventory group.


### Stream proxy usage

```
mygroup_stream_nginx:
  - name: "a_server_with_port_range"
    ports: "{{ range(9000, 9010 +1) |list }}"   # a range can be generated with the range () filter. The +1 is required as range() remove the last element.
    proxy_pass_ip: "192.168.0.123"

  - name: "winrdp"
    ports: [ "3398" ]
    proxy_pass_ip: "my-windows-server.domain.tld"
    proxy_timeout: "1m"
    proxy_timeout: "5m"

  - name: "remapping"
    ports: [ 8080, 8081 ]
    # using an ip, and can also be a dns address
    # the http(s) protocol has no usage for a stream as it is a tcp connection
    proxy_pass_ip: "192.168.3.210"
    # remapping the proxy 8081 port to the backend server 1071 port
    proxy_backend_ports:
      - "port_8081": 1071
```

