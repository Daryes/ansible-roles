# Changelog

## release 2024-07-15

* role app-vaultwarden: New parameter `vaultwarden_db_psql_version` to select the pgsql container version, when activated.  
  New nginx template for the integrated websocket in the main http port.  
  The older template has been renamed to v1.29_max.  
  
* role dns-bind: add capability to search for partial zone records in all groups in the inventory.  
  Supported by the db zones "standard" and "reverse" for the "records_raw" parameter.  
  The behavior is the same as for the apache, nginx or user management roles.  
  
* role sys-packages-update: new parameter to print the list of available updates  
  and the security-only updates are now fully functional on debian.  
  
* role monitoring-grafana: the provisioning tasks are moved to a dedicated main file.  
  Not a real change, just organizing

* role monitoring-grafana-dashboard: mxswat-separator-panel plugin is removed, replaced with an empty text panel.  
  The concerned dashboards are : home.json, node_server_linux.json and node_server_windows.json.  
  
* role monitoring-prometheus: fix a wrong indentation for the `prometheus_job_raw_settings` parameter.  
  It should work as expected.  

