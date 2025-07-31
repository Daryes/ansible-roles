# Ansible role: monitoring-prometheus_agent

### Agents parameters

#### Agent jmx_exporter for Java applications

Website : https://github.com/prometheus/jmx_exporter  

The agent is only copied on the target filesystem under /usr/local/bin, and its default configuration under /etc/prometheus.  
It still must be manually inserted in the java application service.  

The usual syntax for its activation is to change the JVM settings using the java opts environmentvariable.  
Example : 
```
JAVA_OPTS="-javaagent:/usr/local/bin/jmx_prometheus_javaagent.jar=9901:/etc/prometheus/jmx.yml"
```

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_agent_jmx_install | set to yes to install the agent | boolean | no |
| prometheus_agent_jmx_version | Agent version release to install. Do not specify the 'v' in front.<br />Ex: "0.15.0" | "string" | mandatory |
| prometheus_agent_jmx_version_hash | Hash for the archive package file. Syntax follows the hashlib python package.<br />This setting is not required and can be omitted. | "sha256:xyz..." | "" |


#### Java 6 agent
The agent for Java6 is dropped since version 0.19.0.  
It is still installed for the older versions, but will be automatically skipped for the newest.

