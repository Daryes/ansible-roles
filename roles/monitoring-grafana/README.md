# Ansible role: monitoring-grafana

Install grafana on the system using the official repository packages.  
Homepage: http://www.grafana.com


## Restrictions and limitations

The official repository is not activated in the system, Grafana package is downloaded and directly installed.  
While it is required to follow new releases, this method prevents any surprise update.

## Requirements

mandatory role :
* _include-pkg_download_install
* _include-data_move_location


## Parameters


### Mandatory parameters


| Parameter | Description | Type |
| --------- | ----------- | ---- |
| grafana_version | grafana version release to install.<br />Do not specify a "v" in front.<br />Ex: "9.3.1" | "string" |
| grafana_version_hash_deb | Hash for the debian package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." |
| grafana_version_hash_rpm | Hash for the rpm package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." |


### Optional parameters


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| grafana_domain | grafana public domain to access grafana from a browser<br />Should be "service.domain.tld" | "string" | "{{ inventory_hostname }}" |
| grafana_root_url | full public url to access grafana from a browser<br />Protocol and the '/' at the end are required | "string" | "https://{{ grafana_domain }}/" |
| grafana_listen_ip | listen IP.<br />Should be changed to 127.0.0.1 if you're using a reverse proxy on the same server | "string" | "0.0.0.0" |
| grafana_listen_port | listen port | "string" or numeric | "3000" |
| grafana_conf_dir | configuration directory, can be moved on a different filesystem<br />A symlink will be created at the original location | "string" | "/etc/grafana" |
| grafana_data_dir | data directory (plugin, dashboards, ...), can be moved | "string" | "/var/lib/grafana" |
| |
| grafana_system_use_home | The systemd service will prevent the access to the real /home for grafana files.<br />Change this parameter to true if such access is required. | boolean | false |
| grafana_system_use_tmp | Same as the _use_home parameter, but for accessing the real /tmp directory instead of the process' stub.  | boolean | false |
| |
| grafana_plugins_install | additional grafana plugins to install<br />ref: https://grafana.com/grafana/plugins/<br />Ex:<br />- "vonage-status-panel"<br />- "natel-discrete-panel" | list[ "string" ] | [] |
| grafana_plugins_update_all | update all installed plugins - set to yes to activate | boolean | no |
| |
| grafana_config_metrics | activate the internal metrics for prometheus and/or graphite | boolean | no |
| grafana_config_metrics_basic_auth_username | basic auth to require authorization to access the metrics endpoints<br />disabled if empty | "string" | "" |
| grafana_config_metrics_basic_auth_password | password (clear text) for the metrics basic auth | "string" | "" |
| grafana_config_dashboards_versions_to_keep | Number of dashboard versions to keep (per dashboard). Default: 20, Minimum: 1 | numeric | 20 |
| grafana_config_annotations_max_age | Duration to keep the annotations (manual and automatic)<br />Default is 0 to keep them forever, which can consume a lot of space in time.<br />Examples: 6h (hours), 10d (days), 2w (weeks), 1M (month). | "string" | "0" |
| |
| grafana_provisioning_org_name | Main organisation name | "string" | "Main Org." |
| grafana_provisioning_admin_login | local admin user | "string" | "admin" |
| grafana_provisioning_admin_password | local admin password.<br />It is mandatory to change the default password | "string" | "admin" |
| |
| grafana_config_smtp | Activate the smtp support | boolean | no |
| grafana_config_smtp_host | smtp host address or name and smtp port | "string" | "localhost:25" |
| grafana_config_smtp_user | smtp username in case a login is required | "string" | "" |
| grafana_config_smtp_password | smtp password for the username<br />If the password contains # or ; you have to wrap it with triple quotes in addition of the single quotes for the declaration.<br />Ex: `'"""#password;"""'` | "string" | "" |
| grafana_config_smtp_from_address | address information for the FROM sender identity | "string" | "admin@{{ grafana_domain }}" |
| grafana_config_smtp_from_name | name information for the FROM sender identity | "string" | "Grafana" |
| grafana_config_smtp_tls_starttls | startTLS management, uses either :<br /> * force with `MandatoryStartTLS`<br />* ask if supported with `OpportunisticStartTLS` (default)<br />* disable with `NoStartTLS` | "string" | "OpportunisticStartTLS" |
| grafana_config_smtp_tls_skip_verify | If set to yes, it will disable the ssl certificate validation | boolean | no |


**Grafana customization parameters**  

These settings are related to the grafana.ini file.  
Reference: https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/  

Notice: some settings are already managed : domain, http_addr, http_port, data directory, plugins settings, analytics reports, enable_gzip and the metrics settings


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| grafana_config_settings | grafana.ini configuration | list[ object ] | [ ] |
| - | section name in grafana.ini<br />It is the [section] group name in a ini file, without the [ ] | section: "string" | mandatory |
| - | Parameter name under the selected section | param: "string" | mandatory |
| - | Value name for the selected parameter<br />The value is always a string | value: "string" | mandatory |

Examples:
```
  - section: "feature_toggles"
    param: "swaggerUi"
    value: "true"

  - param:  "param2"
    ...
```



**Provisionning parameters : general**

Reference: https://grafana.com/docs/grafana/latest/administration/provisioning/  

The following parameters expects the exact same declaration as in the grafana reference file (in the ref. link)

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| grafana_provisioning_datasources | datasource provisionning | list[ object ] | [] |
| grafana_provisioning_dashboards | dashboard provisioning | list[ object ] | [] |


Examples:
```
grafana_provisioning_datasources:
  apiVersion: 1
  deleteDatasources:
    - name: Graphite
      orgId: 1
  datasources:
    - name: Graphite
      type: graphite
      access: prox
      ...

grafana_provisioning_dashboards:
  apiVersion: 1
  providers:
    - name: dashboards
      type: file
      ...
```

**Provisionning parameters : dashboard transfer**

As the dashboards file storage can be different, this setting will allow to transfer dashboards to grafana config directory.  
The destination will always be relative under "/etc/grafana/provisioning/dashboards/"
If omitted, the source last level directory will be reused (ex: my/dashboard/datawarehouse/*.json => datawarehouse/*.json)

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| grafana_provisioning_dashboards_transferts | dashboard copy | list[ object ] | [ ] |
| - | source path | source: "string" | mandatory |
| - | destination path (relative) | dest: "string" | mandatory |
| - | mode, either "dirsync" for a directory,<br /> or "file" for a single file | mode: "string" | mandatory |
 
Examples: 
```
grafana_provisioning_dashboards_transferts: 
  - source: "/a/directory/on/the/ansible/server"
    dest: "relative/location"
    mode: "dirsync"
  - source: "/another/path/to/a/file.json"
    dest: "another/relative/location"
    mode: "file"

  - source: "/etc/ansible/storage/grafana/dashboards/ex1"
    dest: "ex1"
    mode: "dirsync"
```


**Provisionning parameters : user creation**

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| grafana_provisioning_users | list of users to create | list[ object ] | [ ] |
| - | user login, must be unique | login: "string" | mandatory |
| - | user password (clear text)<br />Can be changed afterwards | password: "string" | mandatory |
| - | user visible name | name: "string" | "{{ this.login }}" |
| - | user email | email: "string" | "{{ this.login }}@localhost" |
| - | global admin status | isadmin: boolean | false |
| - | user organization | orgid: numeric | 1 |
| - | user role in the organization<br />Supported values : "Admin", "Editor", "Viewer" | role: "string" | "Viewer" |




## Usage examples

Installation with a local reverse proxy and prometheus on the same server.  
The dashboards are coming from another role (monitoring-grafana-dashboard)

```
grafana_version: "9.3.1"
grafana_version_hash_deb: ""
grafana_version_hash_rpm: ""

grafana_domain: "grafana.domain.tld"
grafana_listen_ip: 127.0.0.1
grafana_listen_port: 3000

# change data and conf storage location
grafana_conf_dir: "/opt/grafana/conf"
grafana_data_dir: "/opt/grafana/data"

grafana_plugins_install:
  - "vonage-status-panel"
  - "natel-discrete-panel"
  - "mxswat-separator-panel"

grafana_config_settings:
  # required to have the alerts working in the UI panels
  - { section: "unified_alerting", param: "execute_alerts",    value: "true" }
  # swagger API ui - disabled by default
  - { section: "feature_toggles",  param: "swaggerUi",         value: "false" }
  # allow writer to manage folders and teams
  - { section: "users",            param: "editors_can_admin", value: "true" }
  # disable some stuff
  - { section: "security",         param: "disable_gravatar",  value: "true" }
  - { section: "users",            param: "allow_sign_up",     value: "false" }
  - { section: "auth.anonymous",   param: "enabled",           value: "false" }
  # logs
  - { section: "log",              param: "mode",              value: "file" }
  - { section: "log",              param: "level",             value: "warn" }

# change the admin user
grafana_provisioning_admin_login: "another_admin_login"
grafana_provisioning_admin_password: "..."

# add a generic reader user
grafana_provisioning_users:
  - login: "generic"
    password: "reader"

# provisioning
grafana_provisioning_datasources:
  apiVersion: 1
  datasources:
     # create the testdata datasource
    - name: "TestData DB"
      type: testdata
      uid: "testdatadb"

    # prometheus server
    - name: Prometheus
      type: prometheus
      access: proxy
      url: http://localhost:9090
      jsonData:
        timeInterval: 1m
        httpMethod: 'POST'
      isDefault: true
      editable: false
      # this bloc can be removed 
      # if basic auth is not activated on prometheus
      basicAuth: true
      basicAuthUser: "prometheus_user"
      secureJsonData:
        basicAuthPassword: "prometheus_password"


# copy the dashboards to the grafana provisionning directory
grafana_provisioning_dashboards_transferts:
  - source: "/etc/ansible/roles/monitoring-grafana-dashboard/files/dashboards"
    dest: "provisioning"
    mode: "dirsync"

# load the dashboards into grafana
grafana_provisioning_dashboards:
  apiVersion: 1
  providers:
    - name: dashboards
      type: file
      updateIntervalSeconds: 60
      options:
        path: /etc/grafana/provisioning/dashboards
        foldersFromFilesStructure: true
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

