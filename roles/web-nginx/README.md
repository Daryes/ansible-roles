# Ansible role: web-nginx

this role will install nginx, with a default configuration, and one or multiple virtual hosts (vhost)  
It is also able to activate/deactivate modules and existing virtual host.

Some templates are provided, for reverse proxy usage or port streaming.  
The support for http to https redirection is included.


This role also has the capability to search in the whole inventory, looking for a particular variable name based on group membership's, following this format: `<a_group_name>_nginx`.  
As such, it is possible to declare virtual hosts in multiple locations, either globally, or only for a specific group or host.  

Example: a host member of the inventory groups "group1" and "group2" will be scanned for variables named :
* group1_nginx
* group2_nginx

If a variable group9_test_nginx exists for this host, it will be ignored as the host is not a member of group9_test.

This allow to have the main settings for nginx attached to the nginx group, and the application website settings attached to its application group.


## Restrictions and limitations

The role does not allow to manage the default website (or virtualhost). A basic configuration is applied, which should return a 410 http code.  
Any site must be provided through a virtual host with a dedicated domain name.

The provided templates will only activate the https / 443 part if a ssl certificate is provided (self-signed certificate are accepted).

Due to the necessity to support some older clients, the default SSL configuration is valid but might be seen somewhat as lacking.  
If you want to increase the security, just change the ciphers configuration in the `templates/etc/nginx/conf.d/ssl_security.include.j2` file.


## Requirements

mandatory role :  
* _include-pkg_install

The nginx package will be automatically installed using the distribution packages.


## Parameters

### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| nginx_global_vhost_disable | List of virtual host sites to disable.<br />It is the name of each link under `/etc/nginx/site-enabled/` without the .conf extension<br/>Example:<br />- "vhost_name"<br />- "vhost_name2" | list[ "string" ] | [ ] |
| nginx_global_extra_modules | Allow to insert extra modules, with their configuration based on new templates<br />Example:<br />- "templates/etc/nginx/modules-availables/mod-1.conf.j2"<br />- "myapp/templates/module.conf.j2" | list[ "string" ] | [ ] |
| nginx_geoip_access_bitbucket | Include the known Bitbucket IP addresses into the global access IP list<br />Notice: the list in the template might be outdated | boolean | no |
| nginx_global_revproxy_list | List of other reverse proxies placed in front of nginx to trust | list:<br />- { ip: "x.x.x.x",  description: "text" } | [ ] |
| nginx_default_site_listen_ip | listen address (IPv4/v6 or unix socket) for the default website.<br />If a website or a stream server is binded to a specific IP, it might be required to also have an explicit bind setting on all sites. | "string" | "*" |
| nginx_default_site_ssl_cert_pem | path to the ssl certificate public key, in text format.<br />Example: "/etc/ssl/private/path/to/cert.pem" | "string" | "" |
| nginx_default_site_ssl_cert_key | path to the ssl certificate private key, in text format.<br />Example: "/etc/ssl/private/path/to/cert.key" | "string" | "" |


**Basic auth definition**  
Reference:  https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/  

This allow using basic auth on websites. While defined as a global parameter, this is due to nginx requiring a unique user list.  
Each vhost has to activate the basic auth usage in its own configuration, using, for example : 
```
raw_settings:
  - "auth_basic_user_file '/etc/nginx/conf.d/auth.htpasswd';"
```

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| nginx_global_auth_user_list | Insert  one or multiple basic auth credentials for the full nginx instance. | list[ object ] | [ ] |
| - | username | user: "username" | mandatory |
| - | password (crypted hash) | password: "$apr1$crypted hash" | mandatory |
| - | user comment (optional) | comment: "description" | "" |

To generate the crypted passwords, use from the apache2-utils package the command: `htpasswd -n <user>`


---
### Specific parameters: reverse proxy virtual host

Related templates:  
* web-nginx/templates/etc/nginx/sites-available/vhost-template.conf.j2  
 (with http => https redirect, and let's encrypt support)
* web-nginx/templates/etc/nginx/sites-available/vhost-template_no_ssl.conf.j2  
  (only http, no ssl support)

The role will use the template to create one file per server_name,  with the required configuration for a http virtual host working as a reverse proxy using the proxy_pass command.  
Required settings for ssl activation are also integrated.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<group_name>_nginx` | list of virtual host definitions | list[ object ] | [ ] |
| - | domain name | server_name: "www.domain.tld" | mandatory |
| - | target for the proxy pass directive<br />The scheme can be http or https. | proxypass_target: "http://app_listen_ip:port" | mandatory |
| - | template to use for the virtual host.<br />Example: "templates/etc/nginx/sites-available/vhost-template.conf.j2" | site_vhost_template: "string" | mandatory |
| - | domain aliases.<br />Example: [ 'domain2.tld', 'domain3.tld', '...' ] | server_alias: [ "string" ] | [ ] |
| - | listen ip address (v4 or v6) or unix socket.<br />Default is set to '*' for all addresses | listen_ip: "string" | "*" |
| - | alternate log directory.<br />Only specified if different than /var/log/nginx" | site_vhost_log_dir: "string" | "/var/log/nginx" |
| - | additional modules.<br />Same as the nginx_global_modules_activate variable| extra_modules: [ "string" ] | [ ] |
| - | activation status<br />If set to no, the existing virtual host will be deactivated | active: boolean | yes |
| - | ssl certificate. Path to the public certificate in text format.<br />Example: "/etc/path/to/cert.pem" | ssl_cert_pem: "string" | "" |
| - | ssl certificate. Path to the private key in text format.<br />Example: "/etc/path/to/cert.key" | ssl_cert_key: "string" | "" |
| - | Filter access using ip addresses | corp_access: boolean | no |
| - | Raw nginx parameters, taken as-is.<br />Each line must be terminated with a ";" | raw_settings: [ "string" ] | [ ] |

The site_vhost_template parameter can target one of the provided templates, or a custom one from another role.  
In this case, the full path must be specified.  

If you use an external custom template, extra properties can be provided. They will be accessible in the templates under `{{ thisVhost.my_custom_property }}`


**IP access filtering**  

This is useable only when the `.corp_access` parameter is activated.
In this state, all connections will be refused by nginx unless the source IP is a member of one of the following whitelists.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| nginx_global_access_ip_list | list of specific or corporate IP to be allowed to connect | list:<br/>- { ip: "x.x.x.x", description: "text" } | [ ] |
| `<group_name>_web_access_ip_list` | list of ip to allow to connect to the instance.<br />This is complementary to the nginx_global_access_ip_list parameter.<br />While defined per group, is applied for the whole server. | list:<br />- { ip: "x.x.x.x",  description: "site A" }<br />- { ip: "y.y.y.y/24",  description: "Network B" } | [ ] |


Single ip addresses and IP ranges are supported : 10.11.12.13, 10.11.12.0/24, ...  
Notice: nginx accepts a single IP specified as a /32 range, like 10.11.12.13/32, but it might require to remove the /32 range to have it working correctly.


---
### Specific parameters: stream proxy

Related templates:  
* web-nginx/templates/etc/nginx/sites-available/stream-servers.conf.j2

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


The parameter `backend_ports` can be set only for the ports requiring a remapping.  
While it is provided for special cases, the main usage is to keep the same port for the listen port and the proxy target port.


## Usage examples

Both examples are valid for any host member of the "mygroup" inventory group.  


### Reverse proxy usage

```
my_server_name: "vhost.domain.tld"
app_listen_ip: "127.0.0.1"
app_listen_port: "3000"


mygroup_nginx:
  - server_name: "{{ my_server_name }}"
    site_vhost_template: "roles/web-nginx/templates/etc/nginx/sites-available/vhost-template_no_ssl.conf.j2"
    proxy_pass: "http://{{ app_listen_ip }}:{{ app_listen_port }}"
    raw_settings: [ 'client_max_body_size 10M;' ]

```


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


---
# Licence and informations

Provided in the [meta file](meta/main.yml)

