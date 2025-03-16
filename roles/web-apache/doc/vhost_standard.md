# Ansible role: web-apache


### Specific parameters: standard virtual host

Related templates:
* **web-apache/templates/etc/apache2/sites-available/vhost-template.conf.j2**  
  (with http => https redirect, and let's encrypt support)
* **web-apache/templates/etc/apache2/sites-available/vhost-template_no_ssl.conf.j2**  
  (only http, no ssl support)

The role will use the template to create one file per server_name,  with the required configuration for a http virtual host working as a reverse proxy using the proxy_pass command.
Required settings for ssl activation are also integrated.  
The module 'rewrite' is required and must be explitely activated.


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

Given this template is aimed to be generic, its content is limited and you need to specify the main configuration using `raw_settings:`, starting from `DocumentRoot` and `<Directory ...>`

The site_vhost_template parameter can target one of the provided templates, or a custom one from another role.
In this case, the full path must be specified.

If you use an external custom template, extra properties can be provided. They will be accessible in the templates under `{{ thisVhost.my_custom_property }}`


---
## Usage examples

the examples are valid for any host member of the "mygroup" inventory group.


### Standard vhost usage


```
# those are declared aside for convenience
my_server_name: "www.domain.tld"

apache_global_modules_activate: [ 'rewrite', 'ssl', 'headers', 'alias', 'vhost_alias' ]

mygroup_apache:
  - server_name: "{{ my_server_name }}"
    site_vhost_template: "roles/web-apache/templates/etc/apache2/sites-available/vhost-template.conf.j2"
    ssl_cert_pem: /etc/ssl/private/my-cert/my-cert.pem
    ssl_cert_key: /etc/ssl/private/my-cert/my-cert.key
    raw_settings:
      - 'DocumentRoot "/var/www/html/"'
      - '<Directory "/var/www/html">'
      - '  Options None'
      - '  Options -FollowSymlinks +Indexes'
      - '  IndexIgnore .htaccess robots.txt'
      - '  AllowOverride All'
      - '  Require all granted'
      - '</Directory>'

  - server_name: ...
    ...
```

