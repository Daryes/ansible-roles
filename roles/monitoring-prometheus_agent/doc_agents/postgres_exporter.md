# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent postgres_exporter for PostgreSQL 

Website : https://github.com/prometheus-community/postgres_exporter

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_postgresql_install | set to yes to install the agent | boolean | no |
| prometheus_agent_postgresql_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_postgresql_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_postgresql_listen | Agent port and listen ip | "string" | "0.0.0.0:9187" |
| prometheus_agent_postgresql_service_extra_args | Extra parameters for the agent service.<br />Notice: 'web.config.file' and 'auto-discover-databases' are already managed | "string" | "" |
| prometheus_agent_postgresql_user_create | Create the monitoring user | boolean | false |
| prometheus_agent_postgresql_user |  Monitoring user | "string" | "exporter" |
| prometheus_agent_postgresql_password | Monitoring user password.<br />In clear text | "string" | "CHANGE_ME" |
| prometheus_agent_postgresql_auth_sslmode | set to yes to use ssl with PostgreSQL authentification for the exporter | boolean | false |

