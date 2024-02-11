# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent smokeping_prober for network latency

Website : https://github.com/SuperQ/smokeping_prober 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_smokeping_install | set to yes to install the agent | boolean | no |
| prometheus_agent_smokeping_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_smokeping_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_smokeping_listen | Agent port and listen ip | "string" | "0.0.0.0:9374" |
| prometheus_agent_smokeping_service_extra_args | Extra parameters for the agent service | "string" | "" | 
| prometheus_agent_smokeping_targets | list of remote domain or ip to monitor.<br />Ex: [ "server.domain.tld", "1.2.3.4" ] | list[ "string" ] | [ ] |

