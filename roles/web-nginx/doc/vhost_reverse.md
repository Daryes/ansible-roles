# Ansible role: web-nginx


### Specific parameters: reverse proxy virtual host

Related templates:  
* **web-nginx/templates/etc/nginx/sites-available/vhost-template.conf.j2**    
  (with http => https redirect, and let's encrypt support)
* **web-nginx/templates/etc/nginx/sites-available/vhost-template_no_ssl.conf.j2**  
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


Single ip addresses and CIDR ranges are supported : 10.11.12.13, 10.11.12.0/24, ...  
Notice: nginx accepts a single IP specified as a /32 range, like 10.11.12.13/32, but it might require to remove the /32 range to have it working correctly.


---
## Usage examples

The examples are valid for any host member of the "mygroup" inventory group.  


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

