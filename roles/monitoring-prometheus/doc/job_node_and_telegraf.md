
# Job: node exporter

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_node | list of hosts (Linux & Windows) to monitor for the usual system metrics (cpu, ram, network, ...)<br />The default value make use of the ansible automatic group "all", containing all known hosts | list[ "string" ] | [ "{{ groups['all'] }}" ] |
| prometheus_job_node_labels_group_filter | list of group prefixes to search for and add under the label 'roles'<br />Any host member of one of those group will have this extra label containing the groups<br />Ex: [ 'db', 'docker', 'reverse_proxy' ] | list[ "string" ] | [ ] |


# Job: telegraf

Telegraf is a monolithic agent able to replace many prometheus collectors as an alternative.  
Extra tags and other settings are to be set in the configuration of each of telegraf's modules.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_telegraf | List of hosts (any supported system) to monitor for the usual system metrics (cpu, ram, network, ...) and other metrics | list[ "string" ] | [ ] |

If used, Prometheus exporters should be disabled to prevent having multiple collectors running on the same machine, for the same metrics.  


