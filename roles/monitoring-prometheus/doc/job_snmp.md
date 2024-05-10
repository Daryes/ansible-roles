
# Job: SNMP exporter

This job expects the SNMP exporter installed on the same server as Prometheus.  
There is currently no support for a different location.

If the community is not 'public', and/or auth is required, another mibs yaml file must be generated.  
More information here : https://github.com/prometheus/snmp_exporter/issues/762


| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_snmp | list of device to monitor using snmp protocol | list[ object ] | [ ] |
| - | device name | name: "string" | mandatory |
| - | OID target | target: "string" | mandatory |
| - | MIB module name | module: "string" | "" |
| - | additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |

Example:
```
prometheus_job_snmp:
   - name: "some-name"
     target: "1.2.3.4"

   - name: "another"
     target: "device.local"
     module: "custom_mib"
     labels:
       - location: "basement"
```

