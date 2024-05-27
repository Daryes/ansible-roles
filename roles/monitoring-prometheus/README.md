# Ansible role: monitoring-prometheus

Install the Prometheus and Alertmanager servers using the official tar.gz archives.  
The role has the possibility to move the configuration and data to another location, if desired.

Official site : https://prometheus.io  
Binaries : https://prometheus.io/download/  


## Restrictions and limitations

The settings allow to manage multiple pre-configured jobs. A dedicated parameter is also available to create one or more custom jobs.    
In such case, the targets must also be provided, through either the custom job or a custom template. 

Altermanager rules and routing are currently static. The template files must be edited if required.  
Rules: templates/etc/prometheus/alert.rules.j2  
Routing: templates/etc/prometheus/alertmanager.yml.j2  


**Notice related to grafana's dashboards:**  
The dashboards in the monitoring-grafana-dashboard make use of the following custom inventory settings from ansible :
* geo_dc : the datacenter localization (recommended format : [UN/LOCODE](https://unece.org/trade/cefact/unlocode-code-list-country-and-territory) )
* envir_type_short : short name for the application environment (prod, form, test, devl ...)
* domain_name : main dns domain for the servers. Usually the Active Directory or datacenter domain, or the public internet domain.


These facts are transmitted to all jobs through the parameter `prometheus_job_all_labels_static` which uses default value for the 3 inventory settings.  
Set this parameter to `[]` to clear it if you don't plan to use theses dashboards.


## Requirements

mandatory role :
* _include-pkg_install
* _include-exe_download_copy
* _include-data_move_location

Extra system packages required by Prometheus and Alertmanager will be automatically installed.


## Parameters

### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| prometheus_version | Prometheus version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" |
| prometheus_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." |
| prometheus_alertmanager_version | Alertmanager version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" |
| prometheus_alertmanager_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." |


### Optional parameters

#### General parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_listen_ip | Listen IP for Prometheus web UI and API<br />Set to "127.0.0.1" if you're using a local reverse proxy | "string" | "0.0.0.0" |
| prometheus_listen_port | Prometheus listen port | "string" | "9090" |
| prometheus_alertmanager_listen_ip | Listen IP for Alertmanager web UI and API<br />Set to "127.0.0.1" if you're using a local reverse proxy | "string" | "0.0.0.0" |
| prometheus_alertmanager_listen_port | Prometheus listen port | "string" | "9093" |
| |
| prometheus_ssl_cert_pem | ssl PEM certificate to use for https.<br />The file must already be present on the prometheus server.<br />It is not required on prometheus if you're using a reverse proxy.<br />Syntax: "/path/to/cert.pem" | "string" | "" |
| prometheus_ssl_cert_key | ssl private key<br />Same requirements as the _cert_pem parameter.<br />Syntax: "/path/to/cert.key" | "string" | "" |
| |
| prometheus_conf_dir | Prometheus and Alertmanager configuration directory.<br />If changed, a symlink will be placed at the initial location | string | "/etc/prometheus" |
| prometheus_data_dir | Prometheus and Alertmanager data directory.<br />If changed, a symlink will be placed at the initial location | string | "/var/lib/prometheus" |
| |
| prometheus_service_extra_args | Extra parameters for the Prometheus service.<br />Notice: `--web.listen-address`, `--config.file`, `--web.config.file`, `--storage.tsdb.path` and `--storage.tsdb.retention.time` are already managed with dedicated parameters | "string" | "" |
| prometheus_alertmanager_service_extra_args | Extra parameters for the Alertmanager service | "string" | "" |
| prometheus_syslog_history_days | log retention in days for the system logs under `/var/log/prometheus` | numeric | 31 |

If a certificate is not provided, Prometheus and Alertmanager will stay in http mode.


#### Authentication with basic auth

Reference: https://prometheus.io/docs/guides/basic-auth/  

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_auth_basic_users | basic auth for prometheus access<br />If empty, it will not be actived. At least one user must be present for activation | list[ object ] | [] |
| - | User name | user: "string" | mandatory |
| - | Crypted password.<br/>This is a  bcrypt hash, check the reference link for more information | pass_crypted: "string" | mandatory |

Example:
``` 
prometheus_auth_basic_users:
  - user: "username"
    pass_crypted: "$2b$12$hNf2l..."
  - user: "anotherone"
    pass_crypted: "$5..."
```

Notice: this will prevent Prometheus to monitor itself, the scraping parameters `prometheus_scrape_auth_basic_user` & `_password` & `_pass_crypted` must also be defined.


#### Authentication and ssl for exporters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_scrape_auth_basic | Activate basic auth usage to access all exporters | boolean | no |
| prometheus_scrape_auth_basic_user | username to access all exporters | "string" | "" |
| prometheus_scrape_auth_basic_password | Exporters password (in clear text) | "string" | "" |
| prometheus_scrape_auth_basic_pass_crypted | _basic_password content in crypted form, same as basic auth | "string" | "" |
| prometheus_scrape_ssl | activate if the exporters are listening on a https connection | boolean | no |
| prometheus_scrape_ssl_insecure_skip_verify | ignore the validity of the exporter's certificate | boolean | yes |


Notice : these settings are applied on all exporters. Having different users per exporter is currently not supported by the role.   
Notice 2 : if the other parameter `prometheus_auth_basic_users` is activated for basic auth, it will also apply to prometheus and alertmanager self-monitoring.  
To bypass this restriction while not using the basic auth for exporters, set _scrape_auth_basic=no but still fill _scrape_auth_basic_user, _password and _pass_crypted.  


---
## Prometheus jobs parameters

The following documentations are available for the supported exporter :  


* [**Job general and common configuration**](doc/job_general.md)


* [**Job: node exporter and job: telegraf**](doc/job_node_and_telegraf.md)


* [**Job: databases**](doc/job_database.md)


* [**Job: Java JMX**](doc/job_javajmx.md)


* [**Job: Blackbox**](doc/job_blackbox.md)


* [**Job: Pushgateway**](doc/job_pushgateway.md)


* [**Job: Smokeping**](doc/job_smokeping.md)


* [**Job: SNMP**](doc/job_snmp.md)


* [**Job: Grafana**](doc/job_grafana.md)


* [**Job: Custom**](doc/job_custom.md)


## Usage examples

```

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

