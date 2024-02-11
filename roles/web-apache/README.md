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


## Requirements

Mandatory role :
 * _include-pkg_install

Collection :
* community.general

System component :
* find (from findutils)

The apache2/httpd package and findutils will be automatically installed using the distribution packages.


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
| apache_default_site_ssl_cert_pem | Path to the ssl certificate public key, in text format.<br />It will be used as default for all sites not providing their own certificates<br />Example: "/etc/ssl/private/path/to/cert.pem" | "string" | "" |
| apache_default_site_ssl_cert_key | Path to the ssl private key, in text format.<br />Example: "/etc/ssl/private/path/to/cert.key" | "string" | "" |


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
### Specific parameters: virtual host

Related templates:
* web-apache/templates/etc/apache2/sites-available/vhost-template.conf.j2
 (with http => https redirect, and let's encrypt support)
* web-apache/templates/etc/apache2/sites-available/vhost-template_no_ssl.conf.j2
  (only http, no ssl support)

The role will use the template to create one file per server_name,  with the required configuration for a http virtual host working as a reverse proxy using the proxy_pass command.
Required settings for ssl activation are also integrated.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<group_name>_apache` | list of virtual host definitions | list[ object ] | [ ] |
| - | domain name for the virtual host | server_name: "www.domain.tld" | mandatory |
| - | template to use for the virtual host.<br />Example: "templates/etc/apache2/sites-available/vhost-template.conf.j2" | site_vhost_template: "string" | mandatory |
| - | domain aliases.<br />Example: [ 'domain2.tld', 'domain3.tld', '...' ] | server_alias: [ string ] | [ ] |
| - | listen ip address (v4 or v6) or unix socket.<br />Default is set to '*' for all addresses | listen_ip: "string" | "*" |
| - | alternate log directory.<br />Only specified if different than "/var/log/apache2" (or httpd/) | site_vhost_log_dir: "string" | "/var/log/apache2" (debian) or "/var/log/httpd" (rhel) |
| - | additional modules.<br />Same as the apache_global_modules_activate variable | modules_activate: [ "string" ] | [ ] |
| - | modules to disable.<br />Same as the apache_global_modules_disable variable | modules_disable: [ "string" ] | [ ] |
| - | activation status<br />If set to no, the existing virtual host will be deactivated | active: boolean | yes |
| - | ssl certificate. Path to the public certificate in text format.<br />Example: "/etc/path/to/cert.pem" | ssl_cert_pem: "string" | "" |
| - | ssl certificate. Path to the private key in text format.<br />Example: "/etc/path/to/cert.key" | ssl_cert_key: "string" | "" |
| - | Raw apache parameters, taken as-is. | raw_settings: [ "string" ] | [ ] |

Given this template is aimed to be generic, its content is limited and you need to specify the main configuration, starting from `DocumentRoot` and `<Directory ...>`

The site_vhost_template parameter can target one of the provided templates, or a custom one from another role.
In this case, the full path must be specified.

If you use an external custom template, extra properties can be provided. They will be accessible in the templates under `{{ thisVhost.my_custom_property }}`


---
### Specific parameters: reverse proxy

Related templates:  
* web-apache/templates/etc/apache2/sites-available/vhost-template_reverse.conf.j2
 (with http => https redirect, and let's encrypt support)
* web-apache/templates/etc/apache2/sites-available/vhost-template_reverse_no_ssl.conf.j2  
  (only http, no ssl support)

The role will use the template to create one file per server_name,  with the required configuration for a http virtual host working as a reverse proxy using the proxy_pass command.  
Required settings for ssl activation are also integrated.


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<group_name>_apache` | list of virtual host definitions | list[ object ] | [ ] |
| - | domain name for the virtual host | server_name: "www.domain.tld" | mandatory |
| - | target for the proxy pass directive.<br />The scheme can be http or https.<br />Notice: Check the documentation for the proxied application if a `/` is required after the port. | proxypass_target: "http://app_listen_ip:port" | mandatory |
| - | template to use for the virtual host.<br />Example: "templates/etc/apache2/sites-available/vhost-template_reverse.conf.j2" | site_vhost_template: "string" | mandatory |
| - | domain aliases.<br />Example: [ 'domain2.tld', 'domain3.tld', '...' ] | server_alias: [ string ] | [ ] |
| - | listen ip address (v4 or v6) or unix socket.<br />Default is set to '*' for all addresses | listen_ip: "string" | "*" |
| - | alternate log directory.<br />Only specified if different than "/var/log/apache2" (or httpd/) | site_vhost_log_dir: "string" | "/var/log/apache2" (debian) or "/var/log/httpd" (rhel) |
| - | additional modules.<br />Same as the apache_global_modules_activate variable | modules_activate: [ "string" ] | [ ] |
| - | modules to disable.<br />Same as the apache_global_modules_disable variable | modules_disable: [ "string" ] | [ ] |
| - | activation status<br />If set to no, the existing virtual host will be deactivated | active: boolean | yes |
| - | ssl certificate. Path to the public certificate in text format.<br />Example: "/etc/path/to/cert.pem" | ssl_cert_pem: "string" | "" |
| - | ssl certificate. Path to the private key in text format.<br />Example: "/etc/path/to/cert.key" | ssl_cert_key: "string" | "" |
| - | Raw apache parameters, taken as-is. | raw_settings: [ "string" ] | [ ] |

The site_vhost_template parameter can target one of the provided templates, or a custom one from another role.  
In this case, the full path must be specified.  

If you use an external custom template, extra properties can be provided. They will be accessible in the templates under `{{ thisVhost.my_custom_property }}`


## Usage

Variables for a reverse proxy virtual host.  
Valid for any host member of the "mygroup" inventory group.  

```
# those are declared aside for convenience
my_server_name: "vhost.domain.tld"
app_listen_ip: "127.0.0.1"
app_listen_port: "8080"

apache_global_modules_activate: [ 'rewrite', 'headers', 'alias', 'vhost_alias' ]

mygroup_apache:
  - server_name: "{{ my_server_name }}"
    server_alias: []
    site_vhost_template: "web-apache/templates/etc/apache2/sites-available/vhost-template_reverse.conf.j2"
    proxy_pass: "http://{{ app_listen_ip }}:{{ app_listen_port }}"
    modules_activate: [ 'proxy', 'proxy_http', 'proxy_wstunnel' ]
    raw_settings: []

  - server_name: "www.domain.tld"
    site_vhost_template: "web-apache/templates/etc/apache2/sites-available/vhost-template.conf.j2"
    ssl_cert_pem: /etc/ssl/private/myothercert/myothercert.pem
    ssl_cert_key:  /etc/ssl/private/myothercert/myothercert.pem
    raw_settings: |
      DocumentRoot "/var/www/mysite"
      <Directory "/var/www/mysite">
        Options None
        Options +FollowSymlinks +Indexes
        IndexIgnore .htaccess robots.txt
        AllowOverride All
        Require all granted
      </Directory>
    # Notice : the required modules for the website will be activated globally
    modules_activate: [ 'rewrite', 'headers', 'setenvif', 'substitute' ]
 
  - server_name: ...
    ...
```


---
# Licence and informations

Provided in the [meta file](meta/main.yml)

