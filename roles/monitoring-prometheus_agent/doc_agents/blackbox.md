# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent Blackbox

Website : https://github.com/prometheus/blackbox_exporter 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_blackbox_install | set to yes to install the agent | boolean | no 
| prometheus_agent_blackbox_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_blackbox_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_blackbox_listen | Agent port and listen ip | "string" | "127.0.0.1:9115" |
| prometheus_agent_blackbox_conf_extra_args | Extra parameters for the agent service | "string" | "" |
| prometheus_agent_blackbox_conf_tls_insecure_skip_verify | set to "true" to disable validation of remote certificates<br />This is only for http_* and tls_connect probes | "string" | "false" |
| prometheus_agent_blackbox_conf_icmp_payload_size | ICMP packet size, which can be too large on some situations | number | 56 |

