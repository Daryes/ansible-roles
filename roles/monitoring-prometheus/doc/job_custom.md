
# Job: Custom

This parameter will allow to declare any additional configuration into Prometheus. Each line will be taken as-is.  
Ansible usual syntax still apply.

Prometheus reference: [https://prometheus.io/docs/prometheus/latest/configuration/configuration/]()  
Prometheus jobs examples : [https://github.com/prometheus/prometheus/tree/main/documentation/examples]()

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_raw_settings | List of configuration commands for Prometheus<br />Extra labels are supported | list[ "string" ] | [ ] |

Notice: if auth-basic and/or ssl is used on the remote agents, the dedicated settings must also be provided for each custom job.

Examples:
```
# carefull with the ' ' and " "
prometheus_job_raw_settings:
  - '- job_name: "dockerswarm"'
  - '  dockerswarm_sd_configs:'
  - '   - host: unix:///var/run/docker.sock'
  - '     role: tasks'
  - '  relabel_configs:'
  - '   - source_labels: [__meta_dockerswarm_task_desired_state]'
  - '     regex: running'
  - '     action: keep'

  # this is also possible in a multiline string (mind the ' ' at start and end)
  - '- job_name: "..."
       .
       ..
       ...
    '
  # yaml scalar should also apply
  # ref: https://yaml.org/spec/1.2-old/spec.html#id2795688
  - |
    - job_name: "..."
       .
       ..
       ...
    - job_name "another_one"
      ...
```

