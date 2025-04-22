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
| nginx_global_http_header_x_frame_options | Configure the x-frame-option header at a global level. Use "DENY" to prevent any iframe usage.<br />Only the values "SAMEORIGIN" and "DENY" are supported by the current browsers | "string" | "SAMEORIGIN" |
| |
| nginx_global_ssl_protocols | List of allowed SSL/TLS protocols. | "string" | "TLSv1.2 TLSv1.3" |
| nginx_global_ssl_ciphers | List of allowed SSL ciphers by the server.<br /> Please note the parameter "ssl_prefer_server_ciphers" is active. | string | "EECDH+CHACHA20:EECDH+AESGCM :EDH+AESGCM" |
| nginx_global_ssl_ecdh_curve | Ecdh curve selections for SSL. | "string" | "secp521r1:secp384r1:prime256v1" |
| |
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
An advanced example for `raw_settings` usage is available at the end of the documentation for the standard vhost.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| nginx_global_auth_user_list | Insert  one or multiple basic auth credentials for the full nginx instance. | list[ object ] | [ ] |
| - | username | user: "username" | mandatory |
| - | password (crypted hash) | password: "$apr1$crypted hash" | mandatory |
| - | user comment (optional) | comment: "description" | "" |

To generate the crypted passwords, use from the apache2-utils package the command: `htpasswd -n <user>`


---
## Specific parameters: custom hosts

The following documentations are available for the dedicated vhost templates :  

* [**reverse proxy virtual host**](doc/vhost_reverse.md)

* [**standard virtual host**](doc/vhost_standard.md)

* [**stream proxy**](doc/stream.md)


## Usage examples

Examples are available in each specific vhost template


---
# Licence and informations

Provided in the [meta file](meta/main.yml)

