# Ansible role: monitoring-prometheus

Install the Prometheus and Alertmanager servers using the official tar.gz archives.  
The role has the possibility to move the configuration and data to another location, if desired.

Official site : https://prometheus.io
https://prometheus.io/download/


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

Notice: this will prevent Prometheus to monitor itself, the scraping parameters `prometheus_scrape_auth_basic_user` & `_password` must also exist as one element of `prometheus_auth_basic_users`.


#### Authentication and ssl for exporters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_scrape_auth_basic | Activate basic auth usage to access all exporters | boolean | no |
| prometheus_scrape_auth_basic_user | username to access all exporters | "string" | "" |
| prometheus_scrape_auth_basic_password | Exporters password (in clear text) | "string" | "" |
| prometheus_scrape_ssl | activate if the exporters are listening on a https connection | boolean | no |
| prometheus_scrape_ssl_insecure_skip_verify | ignore the validity of the exporter's certificate | boolean | yes |


Notice : these settings are applied on all exporters. Having different users per exporter is currently not managed by the role.   
Notice 2 : if `prometheus_auth_basic_users` is activated for basic auth, it will also apply to prometheus and alertmanager self-monitoring.  
To bypass this restriction,  `prometheus_scrape_auth_basic_user` and _password must be filled and set as one element of `prometheus_auth_basic_users`.  


---
### Prometheus jobs parameters

Most of each job expect a list of hosts for the main parameter.  
This list can be specific hosts or one or multiple ansible groups. Even the ansible special `all` group can be used.  

Example: 

```
prometheus_job_node:
  - "host1"
  - "host2"
  - "{{ groups['tech'] }}"     # <= all servers in the tech group from the ansible inventory
  - "{{ groups['linux'] }}"    # <= all servers in the linux group
  - "{{ groups['windows'] }}"  # <= all servers in the windows group
  - "{{ groups['mysql'] }}"    # <= all servers in the mysql group from the ansible inventory
  - "{{ groups['all'] }}"      # <= all known servers
```

#### Jobs general parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_scrape_interval | global metrics scrape/collect interval | "string" | "1m" |
| prometheus_scrape_timeout | scrape/collect timeout | "string" | "30s" |
| prometheus_scrape_interval_realtime | Dedicated interval for specific cases.<br />Lower than 10s will cause a cpu strain on the servers | "string" | "10s" |
| prometheus_data_retention_time | Data retention, in days | "string" |"15d" |
| prometheus_job_blackbox | list of remote blackbox instances<br /> The blackbox probe is the agent for the certificate, website, icmp, and other jobs<br />If multiple instances are listed, the role does not support specific configurations, the exact same jobs will be executed on all of them (with an extra tag for distinction) | list["string"] | ['127.0.0.1'] |


#### Jobs general labels (or tags)

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_all_labels_static | list of extra labels with a static value set to each jobs<br />Depending of the usage, if the labels are only used for communication with external applications, use instead the parameter '_job_external_labels'<br />Ex:<br />`- tag1: "value"`<br />`- another: "value2"` | list[object] | Cf the notice section |
| prometheus_job_secondary_labels_static | Identical to _all_labels_static. This one can be placed freely in specific groups or host variables | list[object] | [ ] |
| prometheus_job_external_labels_static | list of label inserted only when communicating with external systems (federation, remote storage, Alertmanager)<br />Identical  to _all_labels_static, with the purpose to reduce the amount of labels when they are not mandatory | list[object] | [  ] |
| prometheus_job_all_labels_variable_name | same as _labels_static but instead will use the value as a variable name to retrieve the value from the hostvars for each host, if defined.<br />It will default to "unknown" when the variable is missing on a host | list[object] | Cf the notice section |


Notice : the _static and _variable_name label parameters uses these default values. To change or add other tags, the complete list must be redeclared.  
```
prometheus_job_all_labels_static:
  - corp: "{{ domain_name |default('unknown') }}"
  - geo: "{{ geo_dc |default('unknown') }}"

prometheus_job_secondary_labels_static: []

prometheus_job_all_labels_variable_name:
  # set the environment if available
  - env: "envir_type_short"
```

Also, the ansible_system, ansible_os_family and ansible_distribution facts are already used when available in the jobs.  

And finally : the labels can have duplicates between the _labels_ parameters, the first occurence will be kept, starting with _labels_variable_name having the highest priority, and _all_labels_static the lowest.


---
#### Job: node exporter

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_node | list of hosts (Linux & Windows) to monitor for the usual system metrics (cpu, ram, network, ...)<br />The default value make use of the ansible automatic group "all", containing all known hosts | list[ "string" ] | [ "{{ groups['all'] }}" ] |
| prometheus_job_node_labels_group_filter | list of group prefixes to search for and add under the label 'roles'<br />Any host member of one of those group will have this extra label containing the groups<br />Ex: [ 'db', 'docker', 'reverse_proxy' ] | list[ "string" ] | [ ] |


---
#### Job: database exporter

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_db_mysql | List of host to monitor for MySQL exporter<br />Ex: [ "host", "host2", ... ] | list[ "string" ] | [ ] |
| prometheus_job_db_pgsql | List of host to monitor for PostgreSQL exporter | list[ "string" ] | [ ] |
| prometheus_job_db_mongo | List of host to monitor for MongoDB exporter | list[ "string" ] | [ ] |
| prometheus_job_db_oracle | List of host to monitor for Oracle exporter | list[ "string" ] | [ ] |


---
#### Job: webserver exporter

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_web_apache | list of host to monitor for the Apache webserver exporter | list[ "string" ] | [ ] |


---
#### Job: SNMP exporter

This job expects the SNMP exporter installed on the same server as Prometheus.  
There is currently no support for a different location.

If the community is not 'public', and/or auth is required, another mibs yaml file must be generated.  
More information here : https://github.com/prometheus/snmp_exporter/issues/762


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_snmp | list of device to monitor using snmp protocol | list[ object ] | [ ] |
| - | device name | name: "string" | mandatory |
| - | OID target | target: "string" | mandatory |
| - | MIB module name | module: "string" | "" |
| - | additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |

Example:
```
prometheus_job_snmp:
   - name: "some-name"
     target: "1.2.3.4"

   - name: "another"
     target: "device.local"
     module: "custom_mib"
     labels:
       - location: "basement"
```


---
#### Job: Blackbox websites

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_website | list of website urls to monitor - extra labels are supported | list[ object ] | [ ] |
| - | Site name | name: "string" | mandatory |
| - | Site url | url: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_website_scrape_interval | set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |
 
Example:
```
prometheus_job_website:
   - name: "domain"
     url: "https://domain.tld"

   - name: "corp"
     url: "http://my.corp-domain.tld"
     labels:
       - service: "my"
```


---
#### Job: Blackbox certificates

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_certificate | list of certificates on remote targets to monitor using a tls connection.<br />The website job also retrieve certificate informations when present<br />Use this job mostly for non-https target (like smtp, dns-doh, ...) | list[ object ] | [ ] |
| - | Certificate name | name: "string" | mandatory |
| - | Server/ip and port.<br />Ex: "domain.tld:443" | target: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_certificate_scrape_interval | Set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |


Example:
```
prometheus_job_certificate:
   - name: "domain"
     target: "domain.tld:443"

   - name: "my.corp-domain.tld"
     target: "10.20.30.1:9043"
     labels:
       - service: "my"
```

---
#### Job: Blackbox ICMP

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_icmp | List of remote targets to monitor using icmp<br />Extra labels are supported | list[ object ] | [ ] |
| - | target name | name: "string" | mandatory |
| - | target dns/ip | target: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_icmp_scrape_interval | Set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |


Example:
```
prometheus_job_icmp:
   - name: "domain"
     target: "domain.tld"

   - name: "router"
     target: "10.20.30.1"
     labels:
       - service: "mylabel"
```


---
#### Job: Java JMX

The configuration will search for all variables reusing an inventory group name and ending with '_exporter_jmx'  
For example: `myapp_exporter_jmx`  

A server will be skipped when such variable is found and the server is not a member of the group, and it will not be collected.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<group_name>_exporter_jmx` | Settings for the JMX group | list[ object ] | [ ] |
|   | name of the application or module | name: "string" | mandatory |
|   | main network port of the application or module.<br />Ex: `port: 9090` | port: numeric | mandatory |
|   | port for the prometheus JMX exporter<br />Ex: `exporter_port: 19090`  | exporter_port: numeric | mandatory | 
|   | exporter host, use this in case it must use a different name or ip.<br />Default to the server name<br /> | exporter_host: "string" | "{{ inventory_hostname }}" |
|   | Name of the group label applied to multiple jvm running together | group: "string" | "" |
|   | list of labels to add to the metrics | labels: [ "tag": "value", ... ] | [ ] |

Notice : the elements for this job is a single list (without the - in front) , and not a list of hash.

Example: 
For an application part of an inventory group named `my-app`
```
my-app_exporter_jmx:
  name: "my module"
  port: 9090
  exporter_port: 39090
  group: "my app"
  labels:
    - "my_tag": "no value"
```

---
#### Job: Pushgateway

This configuration is used to collect the [pushgateway acceptor](https://github.com/prometheus/pushgateway)  


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_pushgateway | list of remote pushgateway instances to collect<br />Ex: `[ "localhost", "remote.domain.tld" ]` | list[ "string" ] | [ ] |


---
#### Job: Smokeping

This configuration is used to collect the [smokeping prober](https://github.com/SuperQ/smokeping_prober)  
Notice: the remote hosts to monitor by smokeping must be declared on the smokeping agent side.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_smokeping | List of remote smokeping instance to collect<br />Extra labels are supported | list[ object ] | [ ] |
| - | name of the instance | name: "string" | mandatory |
| - | dns/ip and port of the smokeping instance<br />Ex: `target: "localhost:9374"` | target: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_smokeping_scrape_interval | Set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |


Example:
```
prometheus_job_smokeping:
  - name: "prometheus"
    target: "localhost:9374"

  - name: "outside"
    target: "10.20.30.45:9374"
    labels:
      - service: "mylabel"
```


---
#### Job: Grafana

This configuration is used to collect internal metrics from any Grafana instance.  
Grafana exporter must be activated in its own configuration.  
More information: [https://grafana.com/docs/grafana/latest/setup-grafana/set-up-grafana-monitoring/]()


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_grafana | List of remote grafana instances to collect | list[ object ] | [ ] |
| - | instance name | name: "string" | mandatory |
| - | instance remote address and port.<br />The remote port must always be provided.<br />Ex: "remote-server:3000" | target: "string" | mandatory |
| |
| prometheus_job_grafana_ssl | Activate to connect to all grafana instances using https<br />This cannot be set per instance | boolean | no |
| prometheus_job_grafana_ssl_insecure_skip_verify | disable certificate validity check | boolean | no |


Example:
```
prometheus_job_grafana:
  - name: "myServer"
    target: "localhost:3000"

  - name: "myOtherServer"
    target: "remote-server:53000"


# alternate configuration for ssl

prometheus_job_grafana_ssl: yes
prometheus_job_grafana:
  - name: "myPublicServer"
    # port is always mandatory
    target: "remote-server-proxy:443"
```


---
#### Job: Custom

This parameter will allow to declare any additional configuration into Prometheus. Each line will be taken as-is.  
Ansible usual syntax still apply.

Prometheus reference: [https://prometheus.io/docs/prometheus/latest/configuration/configuration/]()  
Prometheus jobs examples : [https://github.com/prometheus/prometheus/tree/main/documentation/examples]()

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_raw_settings | List of configuration commands for Prometheus<br />Extra labels are supported | list[ "string" ] | [ ] |

Notice: if auth-basic and/or ssl is used on the remove agents, the dedicated settings must also be provided for each custom job.

Examples:
```
# carefull with the ' ' and " "
prometheus_job_raw_settings:
  - '- job_name: "dockerswarm"'
  - '  dockerswarm_sd_configs:'
  - '   - host: unix:///var/run/docker.sock'
  - '     role: tasks'
  - '  relabel_configs:'
  - '   - source_labels: [__meta_dockerswarm_task_desired_state]'
  - '     regex: running'
  - '     action: keep'

  # this is also possible in a multiline string (mind the ' ' at start and end)
  - '- job_name: "..."
       .
       ..
       ...
    '
  # yaml scalar should also apply
  # ref: https://yaml.org/spec/1.2-old/spec.html#id2795688
  - |
    - job_name: "..."
       .
       ..
       ...
    - job_name "another_one"
      ...
```


## Usage examples

```

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

