# Ansible role: app-vaultwarden

This role install [Vaultwarden](https://github.com/dani-garcia/vaultwarden) using its docker image.  

The following backends are accepted :  
* PostgreSQL (provided in the compose file if activated)
* SQLite
* any supported external database. 

PostgreSQL is recommended for production use.  


## Restrictions and limitations

Setting the version parameter to "latest" might not update the deployed version.  
It should either be changed to a specific version or a `docker pull` must be ran on the server.

The PGSQL database is stored under the relative directory `./data` and not as a docker volume.

The main configuration, while provided, has limited templating support.  
If more settings are required, they should be inserted directly in the template, as the deployed file will be overwritten each time the role will be executed.

A template file for nginx is provided, but its content is dedicated to the web-nginx role.


## Requirements

mandatory role :
* _include-compose_deploy

System :  
* docker
* docker-compose


## Parameters

### Mandatory parameters

None.


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| vaultwarden_version | Vaultwarden docker image version to install or update<br />Available tags can be retrieved from [dockerhub](https://hub.docker.com/r/vaultwarden/server) | "string" | "latest" |
| vaultwarden_install_dir | Directory to install the compose yaml file, configuration and data | "string" | "/opt/vaultwarden" |
| vaultwarden_use_compose_service | If activated, will make use of the docker-compose@instance service from the sys-docker role to autostart the service | boolean | no |
| vaultwarden_domain_url | External domain url. Format: `https://sub.domain.dns`<br />The protocol (https) must be present. | "string" | "https://unknown.nowhere.tld" |
| vaultwarden_listen_ip | Listen IP mapping. If a reverse proxy on the same server is not present, it must be changed to `0.0.0.0` | "string" | "127.0.0.1" |
| vaultwarden_listen_port | Listen port mapping to the web application | numeric | 3080 |
| vaultwarden_listen_port_ws | Listen port mapping to the websocket for notifications | numeric | 3012 |
| vaultwarden_admin_token | Access token for the admin interface.<br />If empty or not set, the admin panel is disabled. After initialization, the setting can stays empty to lock the access to the panel.<br />Use this command as a generator:  `openssl rand -base64 48` | "string" | "" |
| |
| vaultwarden_logging_driver_syslog | Set to yes to redirect all logs to syslog. | boolean | no |
| |
| vaultwarden_db | Database type, set to :<br />- `"sqlite"` for a local sqlite file<br />- `"psql"` for a dockerized postgresql companion within the same compose file<br />- `"external"` for any supported remote database | "string" | "sqlite" |
| vaultwarden_db_psql_version | PostgreSQL only: database container version to use | "string" | "16" |
| vaultwarden_db_local_password | PostgreSQL only: database password | "string" | "n8p2dRnPSqxoPea3Y8Bi" |
| vaultwarden_db_external_url | External db only : Full connection string.<br />Supported formats:<br />- Mysql & MariaDB : `mysql://[[user]:[password]@]host[:port][/database]`<br />- PGSQL : `postgresql://[[user]:[password]@]host[:port][/database]` | "string" | "" |
| |
| vaultwarden_smtp_host | smtp remote server - ex: smtp.domain.tld<br />Default is set to localhost to prevent mistakes if not configured | "string" | "localhost" |
| vaultwarden_smtp_security_mode | Security mode, support the following options:<br />- `"starttls"` for port 587 (and 25 if a certificate is available)<br />- `"force_tls"` for port 465<br />- `"off"` for port 25 | "starttls" or "force_tls" or "off" | "starttls" |
| vaultwarden_smtp_port | smtp port to connect to | numeric | 587 |
| vaultwarden_smtp_username | smtp username account | "string" | "" |
| vaultwarden_smtp_password | smtp password | "string" | "" |
| vaultwarden_smtp_from_email | Sender email.<br />Ex: "user@domain.tld" | "string" | "vaultwarden@domain.tld"
| vaultwarden_smtp_from_name | Sender visible name | "string" | "Vaultwarden" |


## Usage examples

This will deploy vaultwarden, using the "latest" version and a PostgreSQL database.  

A template is provided for the web-nginx role. If used, it will be applied with the following configuration.  
Also, this must be defined in the inventory in a group named "app_vaultwarden", which the target server is a member of. Change the `<my group>_nginx` line to use a different group.

```
# name to access vaultwarden
vaultwarden_server_name: vaultwarden.domain.tld


vaultwarden_listen_ip: "127.0.0.1"
vaultwarden_listen_port: 3080
vaultwarden_listen_port_ws : 3012


vaultwarden_db : "psql"
vaultwarden_db_local_password: "m8jqRuMr_GE2lBazfomqcRjvMxYTq4atJbK"
vaultwarden_admin_token: "C3WxUELvZyHpVldRdxwj5C@Fn6KRCl5VVRfD2RCdypSVAV616hG0yDNxdIp+f2PG"


# nginx configuration - adjust the ssl_* path to target an existing certificate on the host
app_vaultwarden_nginx:
  server_name: "{{ vaultwarden_server_name }}"
  site_vhost_template: "app-vaultwarden/templates/etc/nginx/sites-available/vaultwarden.conf.j2"
  proxypass_target: "http://127.0.0.1:{{ vaultwarden_listen_port }}"
  proxypass_notification_target: "http://127.0.0.1:{{ vaultwarden_listen_port_ws }}"
  ssl_cert_pem: "/etc/private/path/to/my/certificate.pem"
  ssl_cert_key: "/etc/private/path/to/my/certificate.key"

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

