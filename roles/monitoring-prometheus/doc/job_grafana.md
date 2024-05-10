
# Job: Grafana

This configuration is used to collect internal metrics from any Grafana instance.  
Grafana exporter must be activated in its own configuration.  
More information: [https://grafana.com/docs/grafana/latest/setup-grafana/set-up-grafana-monitoring/]()


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_grafana | List of remote grafana instances to collect | list[ object ] | [ ] |
| - | instance name | name: "string" | mandatory |
| - | instance remote address and port.<br />The remote port must always be provided.<br />Ex: "remote-server:3000" | target: "string" | mandatory |
| |
| prometheus_job_grafana_ssl | Activate to connect to all grafana instances using https<br />This cannot be set per instance | boolean | no |
| prometheus_job_grafana_ssl_insecure_skip_verify | disable certificate validity check | boolean | no |


Example:
```
prometheus_job_grafana:
  - name: "myServer"
    target: "localhost:3000"

  - name: "myOtherServer"
    target: "remote-server:53000"


# alternate configuration for ssl

prometheus_job_grafana_ssl: yes
prometheus_job_grafana:
  - name: "myPublicServer"
    # port is always mandatory
    target: "remote-server-proxy:443"
```

