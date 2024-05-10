# Ansible role: monitoring-cadvisor_agent

Monitoring agent for analyzing resource usage and performance characteristics of running containers.
Homepage: https://github.com/google/cadvisor

The role uses the official docker image.


## Restrictions and limitations

Cadvisor does not support basic auth nor SSL on the metrics endpoint for prometheus and statd.  
For more information : https://github.com/google/cadvisor/issues/2539  
To alleviate this limitation, the webserver Caddy is used as a companion in a reverse proxy mode, managing the ssl and basic auth configuration.  
More information can be found in the docker-compose template.


If you have cadvisor installed with cadvisor_auth_basic and/or cadvisor_ssl_cert_* settings and you want to deactivate both, the reverse proxy service will be removed from the compose file.  
Then, docker-compose will fail on this situation, as the loss of the service will keep the container running.  
This is a compose limitation.  
For such situation, you need to execute first a "docker-compose down reverse" on the servers with either basic auth or ssl active, using an Ansible ad-hoc command and the shell module.  
Only after the reverse proxy container has been cleaned, you can execute the role to deactivate both basic auth and ssl.

A template for using Traefik instead of Caddy is present and fully working with SSL and/or basic auth.  
No option is available for switching between both, it must be done manually in the role.


## Requirements

mandatory role :
* _include-compose_deploy
* docker and docker-compose must already be installed


## Parameters

### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| cadvisor_version | Image version - format : "x.y.z" - numbers only, without any "version" or "v" in front<br />Ref: https://github.com/google/cadvisor/releases | "string" |


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| cadvisor_compose_dir | Directory to install compose and configuration files | "string" | "/opt/cadvisor" |
| cadvisor_compose_reverse_proxy | Reverse proxy selection between Caddy and Traefik | "caddy" or "traefik" | "caddy" |
| cadvisor_host_listen_ip | Cadvisor external listen IP.<br />If driver="prometheus" =>  _listen_ip should be set to "0.0.0.0"<br />If driver="influxdb" then cadviser will push itself the metric and _listen_ip should be left to localhost | "string" | "127.0.0.1" |
|cadvisor_host_listen_port | Cadvisor external listen port. | "string" | "9135" |
| |
| cadvisor_extra_parameters | Cadvisor extra parameters<br />Ref: https://github.com/google/cadvisor/blob/master/docs/runtime_options.md<br />Notice: `--store_container_labels=xxx` is already managed. | "string" | "--docker_only=true --housekeeping_interval=30s --storage_duration=5m0s" |
| cadvisor_extra_parameters_disable_metrics | Defined metric to skip and not retrieve (single line).<br />* the 'accelerator' metric  is not supported anymore since 2024 | "string" | "--disable_metrics=percpu,sched, tcp,udp,disk,diskIO, hugetlb,referenced_memory, cpu_topology,resctrl" |
| cadvisor_params_store_container_labels | Allow to retrieve and store as labels in the metrics all container labels | boolean | false |
| cadvisor_params_whitelisted_container_labels | Comma separated list of container labels to use in the output metrics.<br />The parameter _store_container_labels must also be set to false | "string" | "" |
| docker_data_dir | Docker data dir in case it has been moved | "string" | "/var/lib/docker" |
| cadvisor_logging_driver_syslog | Special configuration to redirect to syslog and tag the container logs | boolean | no |


**Storage driver settings**  


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| cadvisor_storage_driver | For now, 2 drivers are supported : "prometheus" or "influxdb" | "string" | "prometheus" |
| | 
| cadvisor_storage_driver_prometheus_endpoint | Context root for the metrics endpoint | "string" | "/metrics" |
| |
| cadvisor_storage_driver_influxdb_host_port | Ip and port to the influxdb server | "string" |"10.11.12.13:8086" |
| cadvisor_storage_driver_influxdb_secure | Set to yes to use a https connection to the influxdb server | boolean | no |
| cadvisor_storage_driver_influxdb_db | Influxdb database to use | "string" | "cadvisor" |
| cadvisor_storage_driver_influxdb_user | Influxdb user to access the database. Can be empty | "string" | "" |
| cadvisor_storage_driver_influxdb_password | Password for the Influxdb user. Can be empty | "string" |  "" |
| cadvisor_storage_driver_influxdb_retention_policy | Influxdb retention policy to use. If empty, the default policy will be user | "string " | "" |
| cadvisor_storage_driver_influxdb_polling | Time between each push to the influxdb backend | "string" | "60s" |


**Basic auth and TLS settings**  

Notice: these parameters will only activate for the "prometheus" driver

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| cadvisor_auth_basic | Activate basic auth usage to access the multiple exporters<br />ref: https://github.com/prometheus/exporter-toolkit/blob/master/docs/web-configuration.md | boolean | no |
| cadvisor_auth_basic_user | basic auth user | "string" | "" |
| cadvisor_auth_basic_pass_crypted | crypted password in bcrypt format for the user | "string" | "" |
| |
| cadvisor_ssl_cert_pem | Full path to the certificate pem file | "string" | "" |
| cadvisor_ssl_cert_key | Full path to the certificate key, in crt format.<br />If empty or missing, SSL will not be activated | "string" | "" |


## Usage examples

```
cadvisor_version: "0.49.1"

# not much is need for a prometheus usage
cadvisor_compose_dir: "/server/sys-cadvisor"
cadvisor_host_listen_ip: "0.0.0.0"
cadvisor_host_listen_port: "9135"

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

