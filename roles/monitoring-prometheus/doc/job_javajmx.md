
# Job: Java JMX

The configuration will search for all variables reusing an inventory group name and ending with '_exporter_jmx'  
For example: `myapp_exporter_jmx`  

A server will be skipped when such variable is found and the server is not a member of the group, and it will not be collected.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| `<group_name>_exporter_jmx` | Settings for the JMX group | list[ object ] | [ ] |
|   | name of the application or module | name: "string" | mandatory |
|   | main network port of the application or module.<br />Ex: `port: 9090` | port: numeric | mandatory |
|   | port for the prometheus JMX exporter<br />Ex: `exporter_port: 19090`  | exporter_port: numeric | mandatory | 
|   | exporter host, use this in case it must use a different name or ip.<br />Default to the server name<br /> | exporter_host: "string" | "{{ inventory_hostname }}" |
|   | Name of the group label applied to multiple jvm running together | group: "string" | "" |
|   | list of labels to add to the metrics | labels: [ "tag": "value", ... ] | [ ] |

Notice : the elements for this job is a single list (without the - in front) , and not a list of hash.

Example: 
For an application part of an inventory group named `my-app`
```
my-app_exporter_jmx:
  name: "my module"
  port: 9090
  exporter_port: 39090
  group: "my app"
  labels:
    - "my_tag": "no value"
```

