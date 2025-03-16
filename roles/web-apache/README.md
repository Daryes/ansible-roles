# Ansible role: web-apache


## Description

This role will install apache httpd, and set a default configuration.  
It is also able to activate/deactivate modules and vhost.  

Some vhost templates are provided, mostly for reverse proxy usage.  
The support for http to https redirection is included.


This role also has the capability to search in the whole inventory, looking for a particular variable name based on group membership's, following this format: `<a_group_name>_apache`.
As such, it is possible to declare virtual hosts in multiple locations, either globally, or only for a specific group or host.

Example: a host member of the inventory groups "group1" and "group2" will be scanned for variables named :
* group1_apache
* group2_apache

If a variable group9_test_apache exists for this host, it will be ignored as the host is not a member of group9_test.

This allow to have the main settings for apache attached to the apache group, and the application website settings attached to its application group.


## Restrictions and limitations

The role does not allow to manage the default website (or virtualhost). A basic configuration is applied, which should return a 410 http code.  
Any site must be provided through a virtual host with a dedicated domain name.

The provided templates will only activate the https / 443 part if a ssl certificate is provided (self-signed certificate are accepted).

The role will reload apache2 on vhost changes, but a process restart will also be triggered on any module changes.  


Due to the necessity to support some older clients, the default SSL configuration is valid but might be seen somewhat as lacking.  
If you want to increase the security, just change the ciphers configuration in the `templates/etc/apache2/conf-available/ssl-listen-security.conf.j2` file.


Notice about RHEL / Centos systems :  
Given the lack of enhancement on the rpm package, the directory structure under /etc/httpd is updated to be more alike the Debian structure.  
This is experimental, and some quirks may happen, as the result is not fully identical to Debian.  
For example, the usual modules can not be activated, a default set is already present, with much less modules than the default installation. Update the templates to add more. 
In addition, the following modules have been reimplemented as on-demand modules, inactive by default : ssl, proxy_balancer, proxy_fcgi, dav  
Use those names with `apache_global_modules_activate` to activate them


## Requirements

Mandatory role :
 * _include-pkg_install

Collection :
* community.general

System component :
* find (from findutils)

The apache2/httpd packages and findutils will be automatically installed using the distribution packages.


## Parameters

### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| apache_conf_dir | Apache configuration location, can be moved elsewere using symlink. Ex: "/opt/apache/conf" | "string" | "/etc/apache2" or "/etc/httpd" |
| apache_webroot_dir | Web root directory, can be moved using symlinks. Ex: "/opt/apache/webroot" | "string" | "/var/www" or "/var/lib/httpd" |
| apache_global_vhost_disable | List of virtual host sites to disable.<br />It is the name of each link under `/etc/apache2/site-enabled/` without the .conf extension<br/>Example:<br />- "vhost_name"<br />- "vhost_name2" | list[ "string" ] | [ ] |
| apache_global_modules_activate | Allow to activate apache modules, without the '_module' suffix<br />Example:<br />- "proxy_http"<br />- "proxy_wstunnel" | list[ "string" ] | [ ] |
| apache_global_modules_disable | same as apache_global_modules_activate, but to disable modules | list[ "string" ] | [ ] |
| apache_global_extra_config | extra configuration templates that will be installed and activated.<br />Example:<br />- "myrole/templates/path/to/template.conf.j2"<br />- "/path/to/template2" | list[ "string" ] | [ ] |
| |
| apache_default_site_ssl_cert_pem | Path to the ssl certificate public key, in text format.<br />It will be used as default for all sites not providing their own certificates<br />Example: "/etc/ssl/private/path/to/cert.pem" | "string" | "" |
| apache_default_site_ssl_cert_key | Path to the ssl private key, in text format.<br />Example: "/etc/ssl/private/path/to/cert.key" | "string" | "" |
| |
| apache_global_fcgi_url | Target url for the fcgi proxy module, if activated.<br />Format is : `fcgi://host:port/`<br />The settings are related to "conf-available/conf_mod_fcgi_php-fpm.conf" | "string" | "fcgi://localhost:9000/" |
| apache_global_fcgi_handler_url | fcgi handler allowing to specify a socket or the remote url | "string" | "proxy:unix:/run/php/php-fpm.sock|fcgi://localhost" |


To activate the fcgi module and make use of the custom configuration (globaly), the following parameters must be set in the inventory :
```
apache_global_extra_config: [ 'templates/etc/apache2/conf-available/conf_mod_fcgi_php-fpm.conf.j2' ]

apache_global_modules_activate:
  - 'proxy'
  - 'proxy_fcgi'
  (...)
```


**Module: MPM Event**  

Reference : https://httpd.apache.org/docs/current/mod/mpm_common.html  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| apache_global_mpm_event | Configuration for the mpm_event module | array[object] | - |
| | | StartServers: numeric | 2 |
| | | MinSpareThreads: numeric | 25 |
| | | MaxSpareThreads: numeric | 75 |
| | | ThreadLimit: numeric | 64 |
| | | ThreadsPerChild: numeric | 25 |
| | | MaxRequestWorkers: numeric | 150 |
| | | MaxConnectionsPerChild: numeric | 256 |

To change any value of those parameters, all of them must be defined in the inventory :
```
apache_global_mpm_event:
  StartServers: 2
  MinSpareThreads: 25
  MaxSpareThreads: 75
  ThreadLimit: 64
  ThreadsPerChild: 25
  MaxRequestWorkers: 150
  MaxConnectionsPerChild: 256
```


---
## Specific parameters: custom hosts

The following documentations are available for the dedicated vhost templates :  

* [**reverse proxy virtual host**](doc/vhost_reverse.md)

* [**standard virtual host**](doc/vhost_standard.md)


## Usage examples

Examples are available in each specific vhost template


---
# Licence and informations

Provided in the [meta file](meta/main.yml)

