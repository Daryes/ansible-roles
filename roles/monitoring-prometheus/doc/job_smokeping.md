
# Job: Smokeping

This configuration is used to collect the [smokeping prober](https://github.com/SuperQ/smokeping_prober)  
Notice: the remote hosts to monitor by smokeping must be declared on the smokeping agent side.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_smokeping | List of remote smokeping instance to collect<br />Extra labels are supported | list[ object ] | [ ] |
| - | name of the instance | name: "string" | mandatory |
| - | dns/ip and port of the smokeping instance<br />Ex: `target: "localhost:9374"` | target: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_smokeping_scrape_interval | Set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |


Example:
```
prometheus_job_smokeping:
  - name: "prometheus"
    target: "localhost:9374"

  - name: "outside"
    target: "10.20.30.45:9374"
    labels:
      - service: "mylabel"
```

