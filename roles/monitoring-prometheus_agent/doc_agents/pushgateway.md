# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent Pushgateway

Website : https://github.com/prometheus/pushgateway 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_pushgateway_install | set to yes to install the agent | boolean | no |
| prometheus_agent_pushgateway_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_pushgateway_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_pushgateway_listen | Agent port and listen ip | "string" | "127.0.0.1:9091" |
| prometheus_agent_pushgateway_conf_extra_args | Extra parameters for the agent service | "string" | "" |

