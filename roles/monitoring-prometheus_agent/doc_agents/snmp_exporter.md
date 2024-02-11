# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent snmp_exporter for SNMP traps

Website : https://github.com/prometheus/snmp_exporter

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_snmp_install | set to yes to install the agent | boolean | no |
| prometheus_agent_snmp_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_snmp_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_snmp_listen | Agent port and listen ip | "string" | "0.0.0.0:9116" |
| prometheus_agent_snmp_service_extra_args| Extra parameters for the agent service.<br />Read this page before changing this setting: https://github.com/prometheus/snmp_exporter#configuration | "string" | "--config.file /usr/local/bin/snmp_exporter-archive/snmp.yml"| 
| prometheus_agent_snmp_mibs_deploy_files | list of manually generated snmp.yml files to deploy from the ansible controller to the agent server.<br />The files will be placed under /etc/prometheus/snmp/<br />Ex:<br />`  - "/path/to/local/ansible/server/non_standard_device.yml"`<br />`  - ...` | list[ "string" ] | [ ] |

