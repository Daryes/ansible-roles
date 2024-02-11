# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent smartctl_exporter for hard drive smart informations

Website : https://github.com/prometheus-community/smartctl_exporter

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_smartctl_install | set to yes to install the agent | boolean | no |
| prometheus_agent_smartctl_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_smartctl_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_smartctl_listen | Agent port and listen ip | "string" | "0.0.0.0:9633" |

