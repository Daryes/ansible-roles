
# Prometheus jobs parameters

Most of each job expect a list of hosts for the main parameter.  
This list can be specific hosts or one or multiple ansible groups. Even the ansible special `all` group can be used.  

Example: 

```
prometheus_job_node:
  - "host1"
  - "host2"
  - "{{ groups['tech'] }}"     # <= all servers in the tech group from the ansible inventory
  - "{{ groups['linux'] }}"    # <= all servers in the linux group
  - "{{ groups['windows'] }}"  # <= all servers in the windows group
  - "{{ groups['mysql'] }}"    # <= all servers in the mysql group from the ansible inventory
  - "{{ groups['all'] }}"      # <= all known servers
```

## Jobs general parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_scrape_interval | global metrics scrape/collect interval | "string" | "1m" |
| prometheus_scrape_timeout | scrape/collect timeout | "string" | "30s" |
| prometheus_scrape_interval_realtime | Dedicated interval for specific cases.<br />Lower than 10s will cause a cpu strain on the servers | "string" | "10s" |
| prometheus_data_retention_time | Data retention, in days | "string" |"15d" |


## Jobs general labels (or tags)

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_all_labels_static | list of extra labels with a static value set to each jobs<br />Depending of the usage, if the labels are only used for communication with external applications, use instead the parameter '_job_external_labels'<br />Ex:<br />`- tag1: "value"`<br />`- another: "value2"` | list[object] | Cf the notice section |
| prometheus_job_secondary_labels_static | Identical to _all_labels_static. This one can be placed freely in specific groups or host variables | list[object] | [ ] |
| prometheus_job_external_labels_static | list of label inserted only when communicating with external systems (federation, remote storage, Alertmanager)<br />Identical  to _all_labels_static, with the purpose to reduce the amount of labels when they are not mandatory | list[object] | [  ] |
| prometheus_job_all_labels_variable_name | same as _labels_static but instead will use the value as a variable name to retrieve the value from the hostvars for each host, if defined.<br />It will default to "unknown" when the variable is missing on a host | list[object] | Cf the notice section |


Notice : the _static and _variable_name label parameters uses these default values. To change or add other tags, the complete list must be declared again.  
```
prometheus_job_all_labels_static:
  - corp: "{{ domain_name |default('unknown') }}"
  - geo: "{{ geo_dc |default('unknown') }}"

prometheus_job_secondary_labels_static: []

prometheus_job_all_labels_variable_name:
  # set the environment if available
  - env: "envir_type_short"
```

Also, the ansible_system, ansible_os_family and ansible_distribution facts are already used when available in the jobs.  

And finally : the labels can have duplicates between the _labels_ parameters, the first occurence will be kept, starting with _labels_variable_name having the highest priority, and _all_labels_static the lowest.

