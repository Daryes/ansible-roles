# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent apache_exporter for Apache httpd

Website : https://github.com/Lusitaniae/apache_exporter 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_apache_install | set to yes to install the agent | boolean | no |
| prometheus_agent_apache_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_apache_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_apache_listen | Agent port and listen ip | "string" | "0.0.0.0:9117" |
| prometheus_agent_apache_service_extra_args | Extra parameters for the agent service.<br />add '--insecure' if required for https connection | "string" | ' --host_override="" ' |
| prometheus_agent_apache_server_status | url to the server-status page - localhost should be prefered | "string" | "http://localhost/server-status/?auto" | 

