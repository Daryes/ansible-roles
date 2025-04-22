# Ansible role: sys-docker


Install docker using the official docker repository, and also docker-compose from github.
A "hello world" image will be run at the end to validate the installation.

Some extra configuration are also available :
* move the /var/lib/docker directory to a different location.
* install a docker-compose@.service unit and its /etc/default/docker-compose.template configuration.  
  It is used to start compose services when relying on the "start: always" parameter is not desired or possible.
* install additional rsyslog configuration (if present) for redirecting the logs from a compose file to /var/log/docker/.
* a daily cron cleaning the builder cache (activated as default), and pruning old containers (disabled as default)


## Restrictions and limitations

The docker installation is exclusive with podman, both cannot be installed together


If you want docker-compose to work with the rsyslog redirect, the compose file must have the following logging settings :
```
service:
  <service name>:
    image: ...
    logging:
      driver: syslog
      options:
        tag: docker-{{.Name}}
```
No need to change {{.Name}}, this setting is supported by docker.  

The log file `/var/log/docker/docker-<service name>.log` will be generated.  
If you activated the rsyslog redirect and don't want to use it for a container, simply remove the "logging: ..." lines from the compose file.  


Any changes to the Docker daemon configuration will end with a validation step before restarting any service.  
In case of an error, you can safely correct the settings and validate with the following command :  
`sudo dockerd --validate --config-file '/etc/docker/daemon.json'`


**Warning :** while the role give the possibility to change the /var/lib/docker location, data of an existing installation will not be moved.  
It is meant mostly for fresh installations than an already existing one.  
In such situation, after having ran this role,  all docker containers must be stopped before __manually__, then the docker service must be stopped.  
You can then move (manually) the /var/lib/docker directory over its new location, then restart docker (or the server itself).


## Requirements

mandatory roles :
* _include-pkg_install
* _include-exe_download_copy


## Parameters


### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| docker_compose_version | Compose version release to install.<br />Do not specify the 'v' in front.<br />Ex: "1.15.2" | "string" |


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| docker_data_dir | Directory containing the docker images, containers and volumes. <br />A symlink will be left at the original location if moved.<br />Ex: "/opt/lib-docker" or "{{ project_root }}/sys-docker" | "string" | "/var/lib/docker" |
| docker_image_helloworld | Docker hello world image. Will be pulled and run to validate the installation<br />Set to empty to skip the validation | "string" | "hello-world" |
| docker_compose_version_same_for_all | It is expected the version to install is the same for all servers<br />Set to false if it is not the case to prevent a possible error with a file missing in the cache | boolean | true |
| docker_compose_usrbin_symlink | Create a symlink under /usr/bin to compose binary<br />Use if /usr/local/bin is not in the sudo or root PATH | boolean | false |
| docker_compose_log_redirect | Install the rsyslog redirect configuration for having docker-compose logs redirected to syslog, then a dedicated file | boolean | true |
| docker_compose_log_dir | directory where rsyslog will store the docker-compose log files | "string" | "/var/log/docker" |
| project_root | root directory where each docker-compose.yml is located in a subdir.<br />Used by the optional docker-compose@.service unit | "string" | "/opt" |
| | | | |
| docker_cron_clean_build_cache | Daily cron job for cleaning build cache, set to yes to activate | boolean | yes |
| docker_cron_clean_container_stopped | Daily cron job for cleaning stopped containers, set to yes to activate | boolean | no |
| docker_proxy_http_url | General proxy url, can be used for registry procies.<br />The port must always been present.<br />Ref: https://docs.docker.com/config/daemon/systemd/#httphttps-proxy<br />Ex: "http://proxy.example.com:80" | "string" | "" |
| docker_proxy_https_url | General https proxy url. Same as _proxy_http_url.<br />Ex: "https://proxy.example.com:443" | "string" | "" |
| docker_proxy_no_proxy | Comma separated list of domains that will skip the proxy.<br />The localhost address is already present. | "string" | "" |
| | | | |
| docker_config_ipv6 | ipv6 usage for containers, set to 'yes' to activate | boolean | no |
| docker_config_registry_mirrors | List of mirrors or personal registries.<br />The https:// scheme must always be present.<br />Ex: `[ "https://myregistry.domain.tld", "https://myregistry2:5002", ... ]` | list[ "string" ] | [ ] |
| docker_config_insecure_registries | List of registries not using TLS.<br />Ex: `[ "http://myregistry.domain.tld:80", "http://myregistry2:5002", ... ]` | list[ "string" ] | [ ] |
| docker_config_log_driver | Docker log driver, can be journald, syslog, ... - the driver 'json-file' is the default | "string" | "json-file" |
| docker_config_log_opts | log driver opts, all the json parameters must be on a single line between `' '`<br />Each property and value are expected to be strings between `" "`<br />Some parameters can be incompatibles with drivers, for example using journald driver does not support "max-size" & "max-file" parameters  | "string" | '"max-size": "10m", "max-file": "3"' |
| docker_config_log_retention_days | Number of days the logs are kept and rotated when using the rsyslog redirector | numeric | 14 |


**Docker networking configuration**

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| docker_config_default_address_pools | Network address pool used for internal networks<br />If empty, docker default configuration will apply | list[ object ] | [ ] |
| - | The full network range. Ex: "172.30.0.0/16" | base: "string" | mandatory |
| - | The size for each single network size generated from the full range.<br />If set as a string, docker will raise an error.<br />Ex: 24 | size: numeric | mandatory |

Example.
```
docker_config_default_address_pools:
  - base: "172.30.0.0/16"
    size: 24

  - base: "172.31.0.0/16"
    size: 24
```


## Usage examples

Set the docker-compose version and move the lib directory. Everything else uses the default values.

```
docker_compose_version: "2.6.1"

# move /var/lib/docker elsewhere
project_root: "/server"
docker_data_dir: "{{ project_root }}/sys-docker"

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

