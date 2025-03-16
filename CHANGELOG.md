# Changelog

## release 2025-03-16 : ansible-core v2.16

**BREAKING:**  
* Global switch to ansible-core v2.16  
  All roles are still fully compatible with older ansible-core versions, but a warning will now be visible when using specific shell commands.  
  This is due to the `warn=false` now raising a blocking error on `shell:` and `command:` modules, since ansible-core v2.14+  
  See the dedicated section for switching back and forth between ansible versions.  
  Please note that in the future, the compatibility might broke.  
  
* role web-nginx: the template `vhost-template.conf` has been renamed to `vhost-reverse.conf`  
  The definitions in the inventory should switch to the `vhost-reverse.conf` template.  
  A symlink is present for the role compatibility, there is no urgency to make the switch, but note the symlink will be removed on a future time.  
 
  
**CHANGES:**
* role web-apache:  
  - readme : split the documentation for the templates in dedicated files.
  - mod_expire: added a default expiration value for the 'image/webp' type.

* role web-nginx:  
  - readme : split the documentation for the templates in dedicated files.
  - explicit installation of the stream module `nginx-mod-stream` for RHEL systems.
  - new global template `global_websocket_map.conf` for a correct handling when opening and closing a websocket.
  - new virtual host template `vhost-standard.conf` and the _no_ssl variant expecting a full directory configuration.

* role sys-docker: logrotate config - the rotation will occur even for empty files.

* role _include-pkg_install:  
  - new parameter `arg_install_force_all` to force the full remove/install/update sequence even if the packages are already installed.
  - new parameters `arg_install_deb_repo` and `arg_install_rpm_repo` replacing the '*_ppa_repo' and '*_yum_repo' parameters.  
    The old parameters are still supported, just not visible anymore.

* role sys-packages-update: readme - clarity on the direct usage of apt and dnf, and about the reboot sequence.

* role ntp-chrony: normalization of the handler names and definition.

* role monitoring-prometheus_agent:  
  - readme - add the missing `_auth_basic_*` parameters to the readme. 
  - typo fix in the installation task for the snmp agent.

* role monitoring-prometheus:  
  - system flag activated for the user group creation.
  - removal of the requirement installation which was not used.

* role monitoring-grafana: explicit parameter `grafana_config_dashboards_versions_to_keep` for the history of dashboard versions.

* role monitoring-grafana-dashboard:
  - dashboard home - fix the alignement and use the environment full name instead of the single char in the server table list.
  - dashboard node_server_linux - show the shared memory usage.

* role db-postgresql: use the 'libpq5' package name with the explicit version on debian.

* role dns-bind handlers - removal of the explicit full path for the bind commands due to being moved from /usr/sbin to /usr/bin


## release 2025-01-15

* role web-nginx: vhost templates - the proxypass section is now optional and will remove the default location if not set.  
  This allow the template to be reused for a standard website using the `raw_settings` parameter

* role db-mysql: new empty template for logrotate to handle system differences.
  The role also now use the xtom.ee mirror

* role app-vaultwarden: fix for the ADMIN_TOKEN in the config file when special chars were used.  
  The previous older image is also removed when updating

* roles certificate-generate* (all): new parameters for the 'locality', 'province', 'key usage' and 'key usage extended' when they were missing

* role _include-data_move_location: readme additions about symlinks and existing data.

* role _include-compose_deploy: add capability to remove old docker images when upgrading the version.
  the value of the _compose_force_pull_always argument is also printed in the execution as an item


## release 2024-08-15

* role monitoring-prometheus: fix for the CPU_Stalled alerts being too aggressive and firing when not necessary.

* role monitoring-prometheus: updated the system label 'os_desc' to implement the ansible_distrib_version fact when LSB is not installed

* role sys-packages-update: set the update list printing to off as default value.  
  The update list print parameter has also been added to the readme  

* role app-vaultwarden: activate the new managed_by_ansible marker in _include-compose_deploy

* role _include-compose_deploy: new parameter to create an empty file MANAGE_BY_ANSIBLE used as a visual marker

* role dns-bind: fix in the reverse-template.db zone with a missing endif, rendering the template unusable


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

