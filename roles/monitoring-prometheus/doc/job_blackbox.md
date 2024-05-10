
# Jobs: Blackbox

## Job: Blackbox websites

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_website | list of website urls to monitor - extra labels are supported | list[ object ] | [ ] |
| - | Site name | name: "string" | mandatory |
| - | Site url | url: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_website_scrape_interval | set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |
 
Example:
```
prometheus_job_website:
   - name: "domain"
     url: "https://domain.tld"

   - name: "corp"
     url: "http://my.corp-domain.tld"
     labels:
       - service: "my"
```


## Job: Blackbox certificates

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_certificate | list of certificates on remote targets to monitor using a tls connection.<br />The website job also retrieve certificate informations when present<br />Use this job mostly for non-https target (like smtp, dns-doh, ...) | list[ object ] | [ ] |
| - | Certificate name | name: "string" | mandatory |
| - | Server/ip and port.<br />Ex: "domain.tld:443" | target: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_certificate_scrape_interval | Set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |


Example:
```
prometheus_job_certificate:
   - name: "domain"
     target: "domain.tld:443"

   - name: "my.corp-domain.tld"
     target: "10.20.30.1:9043"
     labels:
       - service: "my"
```


## Job: Blackbox ICMP

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| prometheus_job_icmp | List of remote targets to monitor using icmp<br />Extra labels are supported | list[ object ] | [ ] |
| - | target name | name: "string" | mandatory |
| - | target dns/ip | target: "string" | mandatory |
| - | Additional labels | labels: [ "label": "value", "label2": ... ] | [ ] |
| |
| prometheus_job_icmp_scrape_interval | Set a different scrape/collect interval<br />The default value reuse the global scrape interval. | "string" | "{{ prometheus_scrape_interval }}" |


Example:
```
prometheus_job_icmp:
   - name: "domain"
     target: "domain.tld"

   - name: "router"
     target: "10.20.30.1"
     labels:
       - service: "mylabel"
```

