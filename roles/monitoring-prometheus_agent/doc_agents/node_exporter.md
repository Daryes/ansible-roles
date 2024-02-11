# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent node_exporter for Linux systems 

Website : https://github.com/prometheus/node_exporter/ 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_node_install | set to yes to install the agent | boolean | yes |
| prometheus_agent_node_version| Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_node_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_node_listen | Agent port and listen ip | "string" | "0.0.0.0:9100" |
| prometheus_agent_node_service_extra_args | Extra parameters for the node agent service | "string" | "--no-collector.infiniband<br />--no-collector.tapestats<br />--collector.logind<br />--collector.sysctl<br />--collector.sysctl.include=fs.file-nr<br />--collector.sysctl.include=fs.dentry-state<br />--collector.sysctl.include=fs.aio-nr<br />--collector.sysctl.include=fs.aio-max-nr<br />--collector.sysctl.include=fs.inode-state" |

