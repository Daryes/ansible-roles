# Ansible role: monitoring-grafana-dashboard

Customized Prometheus dashboards for Grafana.  
Intended to be deployed through Grafana provisioning mechanisms.


## Usage

This role does not have any task or action for ansible, its purpose is only to store the dashboard definitions as static files.  
The main grafana role will manage and sync those files using the settings in the ansible inventory.


## Requirements

The following Grafana plugins are used by the dashboards :  
* natel-discrete-panel 


## Restrictions and limitations

### Generating an export file

The json export file is currently generated using the current dashboard options => JSON model => ctrl-a and copy the full selection


### Windows carriage return

The files are stored as Windows text files, using the usual CRLF as carriage return. Unless you're using a Linux desktop.  
This is due to the json export file copied from the browser, and so will use the current system behavior.  
If required, make use of the dos2unix and unix2dos commands.


### Datasource with generated UID

Grafana has the habit to replace the datasource names by an internal ID. This has the side effect to link the dashboards to the server used to create them.  
It can be seen when opening an imported dashboard, an error will be raised pointing to an unknown datasource.  
For more information : https://github.com/grafana/grafana/issues/10786

To alleviate this issue, all the dashboard have a variable declaration named `dsdashboard`  
It must be used in all the panels as their datasource

Some panel can pass through this, especially for those which do not use a datasource. Yet, the setting will still be here.
To fix this, the datasource ID must be replaced in the json file.  
Open it with the preferred editor and search for `datasource` :
```
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
```
Here is an internal ID.  

It can be replaced either with : 
* a dashboard variable, like `${DS_PROMETHEUS}` or `${dsdashboard}`
* the visible name for the datasource. Pay attention to the letter case, `prometheus` is not the same as `Prometheus`

An useful command to replace an internal ID by the variable `dsdashboard` :
```
# search for "uid": "PBFA97CFB590B2093"
# and replace it globally with "uid": "${dsdashboard}"
#
sed -i 's#"uid": "PBFA97CFB590B2093"#"uid": "${dsdashboard}"#g' *.json
```

Notice: keep the single quotes, the complete search and replace chain must be between ' ' and not " "


# Licence and informations

Provided in the [meta file](meta/main.yml)

The following prometheus dashboards for Grafana are created by :  
* Mysql, Galera, MongoDB : Percona (https://github.com/percona/grafana-dashboards)
* PostgreSQL : Lucas Estienne (https://github.com/lstn/misc-grafana-dashboards)

