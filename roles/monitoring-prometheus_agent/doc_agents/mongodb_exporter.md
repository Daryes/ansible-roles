# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent mongodb_exporter for MongoDB

Website : https://github.com/percona/mongodb_exporter 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_mongodb_install | set to yes to install the agent | boolean | no |
| prometheus_agent_mongodb_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_mongodb_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_mongodb_listen | Agent port and listen ip | "string" | "0.0.0.0:9187" |
| prometheus_agent_mongodb_service_extra_args | Extra parameters for the agent service | "string" | "" |
| prometheus_agent_mongodb_connuri | MongoDB full URI connection string | "string" | "mongodb://user:pass@127.0.0.1:27017/admin?ssl=true" |

