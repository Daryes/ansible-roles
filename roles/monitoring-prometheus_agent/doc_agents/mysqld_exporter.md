# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent mysqld_exporter for MySQL / MariaDB

Website : https://github.com/prometheus/mysqld_exporter 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_mysql_install | set to yes to install the agent | boolean | no |
| prometheus_agent_mysql_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_mysql_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_mysql_listen | Agent port and listen ip | "string" | "0.0.0.0:9104" |
| prometheus_agent_mysqld_service_extra_args | Extra parameters for the agent service | "string" | "" |
| prometheus_agent_mysql_user_create | Create the monitoring user | boolean | false |
| prometheus_agent_mysql_user | Monitoring user | "string" | "exporter" |
| prometheus_agent_mysql_password | Monitoring user password.<br />In clear text | "string" | "CHANGE_ME" |
| prometheus_agent_mysql_socket | Unix socket for communication to create the monitoring user | "string" | "/var/run/mysqld/mysqld.sock" for debian<br />"/var/lib/mysql/mysql.sock" for rhel |

