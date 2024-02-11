# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent oracledb_exporter for Oracle DB

Website : https://github.com/iamseth/oracledb_exporter 

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_oracledb_install | set to yes to install the agent | boolean | no |
| prometheus_agent_oracledb_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_oracledb_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |
| prometheus_agent_oracledb_listen | Agent port and listen ip | "string" | "0.0.0.0:9161" |
| prometheus_agent_oracledb_service_extra_args | Extra parameters for the agent service<br />Notice: 'web.config.file' and 'default.metrics' are already managed | "string" | "--log.level error" |
| prometheus_agent_oracledb_dsname | Oracle datasource connection string | "string" | "read_only_user/password@//myhost:1521/service"  |
| prometheus_agent_oracledb_orahome | Oracle local client directory | "string" | "/u01/app/oracle/product/19.0.0/dbhome_1" |
| prometheus_agent_oracledb_oralib | Oracle client libraries  | "string" | {{ prometheus_agent_oracledb_orahome }}/lib" |

