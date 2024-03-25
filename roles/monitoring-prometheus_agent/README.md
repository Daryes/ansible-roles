# Ansible role: monitoring-prometheus_agent

Install multiples Prometheus exporter agents using the official archives from github.

## Restrictions and limitations

Some settings, like basic auth and the use of SSL is activated globally. Options for a per-agent activation are not implemented.

Each agent expects the listen port defined in the networking listen parameter.  
While the role gives the possibility to change the port numbers, it is recommended to keep the default value for each agent.  

The following agents are not fully tested : oracle, mongodb, smartctl, snmp.  
Some configuration customization might not be present while required to have the agent working correctly


Notice : the installation flag is enabled by default for the node_exporter, while deactivated for all others agents.

## Requirements

mandatory role :
* _include-exe_download_copy
* _include-pkg_install

Extra system packages required by the agents will be automatically installed when required.

All agent installation related to an external module, like a database engine, expects them to be already installed and running.  
Otherwise the installation will not be able to complete.


## Parameters


### Mandatory parameters

None.


### Optional parameters


**General parameters**  

These settings are applied to all agents

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_syslog_history_days | log retention in days for the system logs under /var/log/prometheus | numeric | 15 |
| prometheus_agent_auth_basic | Activate basic auth usage to access the multiple exporters metrics | boolean | no |
| prometheus_agent_auth_basic_user | username for basic auth.<br />Disabled if empty| string | "" |
| prometheus_agent_auth_basic_pass_crypted| Password for basic auth.<br />Password is crypted with bcrypt.<br />Ref: https://github.com/prometheus/exporter-toolkit/blob/master/docs/web-configuration.md | string | "" |
| prometheus_agent_version_same_for_all | It is expected each agent version selected will be the same for all servers<br />Set to false if it is not the case to prevent a possible error with a file missing in the cache when one server uses a specific version | boolean | true |
| prometheus_agent_ssl_cert_pem | ssl PEM certificate to use for https<br />The file must already be present on the server.<br />Syntax: "/path/to/cert.pem" | "string" | "" |
| prometheus_agent_ssl_cert_key | ssl private key.<br />Same requirements as the _cert_pem parameter.<br />Syntax: "/path/to/cert.key" | "string" | "" |

If a certificate is not provided, the agents will default to standard http.

To generate a crypted password with BCRYPT, you need one of the supported utilities, like htpasswd from apache.  
Then, use this command to have an interactive prompt and get the hashed password.
```
htpasswd -nBC 10 "" | tr -d ':'
```


### Agents parameters

See the dedicated documentation for each agent:

* [Agent Blackbox](doc_agents/blackbox.md)  
  for dns, websites, certificates, ports presence, ... metrics

* [Agent Pushgateway](doc_agents/pushgateway.md)  
  receiver for pushing metrics

* [Agent apache_exporter](doc_agents/apache_exporter.md)  
  for the Apache http webserver

* [Agent jmx_exporter](doc_agents/jmx_exporter.md)  
  java agent for Java applications

* [Agent mongodb_exporter](doc_agents/mongodb_exporter.md)  
  for MongoDB databases

* [Agent mysqld_exporter](doc_agents/mysqld_exporter.md)  
  for MySQL and MariaDB databases

* [Agent node_exporter](doc_agents/node_exporter.md)  
  for linux systems

* [Agent oracledb_exporter](doc_agents/oracledb_exporter.md)  
  for Oracle databases

* [Agent postgres_exporter](doc_agents/postgres_exporter.md)  
  for PostgreSQL databases

* [Agent smartctl_exporter](doc_agents/smartctl_exporter.md)  
  for smart information from hard drives

* [Agent smokeping_prober](doc_agents/smokeping_prober.md)  
  for more precise network latency monitoring

* [Agent snmp_exporter](doc_agents/snmp_exporter.md)  
  for retrieving metrics from SNMP traps


## Usage examples

Aside the node_agent, all others agents are not activated. Their _install parameter must be set to yes, and the version set  to have them installed.  
The usual case is to create a group in ansible for the target servers, then fill in the new group the settings for the requested agent.

For example, to install the postgre agent on the PostgreSQL servers, create a group `db_postgresql` containing the targeted servers.  
In this group, add the following settings taken from the parameter list
```

prometheus_agent_node_version: "1.5.0"

prometheus_agent_postgresql_install: yes
prometheus_agent_postgresql_version: "0.15.0"

# try to create the user if missing.
# This might raise some errors, so switch the setting and create it manually if needed
prometheus_agent_postgresql_user_create: yes

# replace the user and password as desired
prometheus_agent_postgresql_user: "exporter"
prometheus_agent_postgresql_password: "{{ myvault.monitoring.postgresql.password }}"
```

This will install and start the PostgreSQL exporter only on the servers members of the `db_postgresql` group.

Proceed in the same way for all the others.

The next step is on prometheus server's side, to either activate the related job for each agent or create one.


## Add a new agent to this role

A lot of setting are very similar, if not identical, and centralizing the agents allows to manage them.  
But the number of agent does not help to simplify the task.  

To add a new agent : 
* Select the name you'll use : probe is used for agents which connect to multiple remote targets.  
  Exporter is used for agents collecting data from a single local service or a system's module.
* under `vars/main/` add a new configuration file for the agent.
  Use one of the already existing file as a template.
* under `tasks/agents/` add a new task file for the agent, similar to one of the others.
  The task must at least handle : 
    - Installing the requirements (this part can be skipped if there are none).
    - Installing the agent.  
      Using a deb or rpm file is fine, but those from the distribution's official repos are frequently outdated.  
      An archive directly from the maintainer's repo is prefered.  
    - Installing the service and configuration files.
    - Stopping the service on change (no handlers here).
    - Starting the service (always).
* under `templates/` add the necessary templates.  
  Usually systemd/, default/, and maybe under var/lib/prometheus/
* in `tasks/main.yml` add a line in the other exporters' sequence to load the dedicated task for the new agent.
* in `default/main.yml` add the related settings for the new agent

Done, test the installation, then, update the README.


# Licence and informations

Provided in the [meta file](meta/main.yml)

