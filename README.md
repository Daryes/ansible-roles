# Personnal Ansible roles

Personal roles for Ansible, battle-tested and generic enough for supporting both Debian and RHEL based distributions.  
While ready to be used, some of them might not be inclined to your tastes, or too opiniated. Some use-cases might also be missing, mostly because they weren't encountered.   
Still, the approach used on many of them is highly different than what is usually done, pushing Ansible more into factorization, reusability and a more intuitive inventory.  

As the community helped me a lot in discovering and mastering Ansible, I though it was time to give back, hence the new toys, and more specifically new ideas, I hope.  

Happy automation.

Notice : if you need a playground but don't have one aside WSL or a single server with some ram, get a look to my other tool, the Ansimulator.

Last thing : english is not my native language, so please bear with any mistake.


A changelog [is available](CHANGELOG.md).  


## Usage and notice

Most of the roles will list their requirements, but some collections might be missing from time to time.  
Basically, ensure those 3 are present : `community.general` `ansible.utils` and `ansible.netcommon`  
Also, a requirement file is provided here under roles/requirements.yml to be used with the `ansible-galaxy` command.


**Presence of symlink**  
Many roles make use of symlinks to simplify the compatibility between Debian and RHEL systems.  
Due to this, using "git clone" or the tar format on a Linux system is prefered to keep the symlinks.


**Supported ansible versions**  
The current roles are structured for Ansible v2.9 (don't ask), but are also valid and tested up to Ansible v2.15.  
Given the major incompatibility with the warn=false parameter, they were altered to ensure a simple grep+sed would make them working correctly.  
If you are using Ansible v2.14+, they can be made compatible with the following one-liner :  
```
egrep -ri '^ *  warn: .*' /etc/ansible/roles/*/{handlers,tasks,vars}/ | cut -d ':' -f1 |sort -u |xargs --no-run-if-empty sed -i '/^ *  warn: .*/d'
```
Adjust the path at the start if the roles are not under `/etc/ansible/roles/`  
If you want to verify which files will be altered, remove the `|xargs ` part up to the end of the command.


# Available roles

## Function roles

These roles are not meant to be used directly, but called by another role passing the required parameters.  
They took root from the recurring need to declare the same set of tasks in different roles, or preventing any optimization due to the amount of work required on each role.  

While on it, the infamous serial by group has been cracked, with a role here generating the correct definition to loop onto.

| Role | Description |
| - | - | 
| <pre>_include-compose_deploy</pre> | Allow to install a docker-compose file from a template, pull the images and start the services | 
| <pre>_include-data_move_location</pre> | Move a directory and its content to another location, leaving a symlink behind. Used mostly for directories under /etc/ and /var/lib/ |
| <pre>_include-exe_download_copy</pre> |  Download and copy an executable or archive file, like for example the phar or docker-compose binary.<br />Manage a file cache on the Ansible server, to reduce load and for air-gapped environments |
| <pre>_include-pkg_download_install</pre> | Download and install a deb or rpm package file.<br />Also manage a file cache on the Ansible server. |
| <pre>_include-pkg_install</pre> | Install or uninstall one or multiple packages using the package manager.<br />Support the activation of external repositories<br />Make also use of the `package_facts` module to heavily reduce the execution time. |
| <pre>_include-serial_over_hostgroups</pre> | Generate from a list of multiple servers a variable containing the servers grouped by the desired indice. When coupled with a conditional, this gives the seeked `serial by group` execution currently not available in Ansible. |


## Roles with their settings anywhere in the inventory

These roles support a more complex declaration in the inventory, using the following format : `<group name>_suffix`  
The suffix is specific to each role, the group name is simply an inventory group the server is a member of.  
This gives the option to not having to rebuild the role when you want to make use of it for another application, just adding the full configuration in the inventory will be enough.  
For example, you can declare multiple users in different groups, the dedicated role will manage all of them.

A caveat : as those roles parse Ansible's facts to assemble the final configuration, they are extremely sensible to any typo, mistake or invalid declaration.  
A validation task is present to trap most of them, but will return an error on multiple lines, due to Ansible limitations. The error will always have the real cause within, but must be pasted and split in an editor to be readable.

| Role | Description |
| - | - | 
| <pre>sys-user_mngt</pre> | User management, support creating groups, users, and authorizing ssh keys.<br />Supports also user and group deletion, only through a global setting. |
| <pre>web-nginx</pre> | Install nginx, and manage virtual hosts and / or stream declarations.<br />Templates are provided, but also support external templates for more complex virtual hosts. |
| <pre>web-apache</pre> | Install Apache http server, with the same virtual host capabilities as the nginx role.<br />RHEL systems are supported, but their configuration will be heavily altered to be more alike the Debian structure, using the *-available and *-enable directories and symlinks. |
| <pre>db-mysql</pre> | Install and configure MariaDB using the official repository, with option to move the configuration and data directories<br />Also supports creating databases and users in the inventories. |
| <pre>db-postgresql</pre> | Install and configure PostgreSQL using the official repository, with the same capabilities as the db-mysql role. |

## Roles for SSL certificates

Basically a full working PKI, focused on having an internal CA and generating any required service certificate.  
Certificates presented as files or variable contents as sources are both supported.  
Notice : they all require the `community.crypto` collection, which can also be installed on Ansible v2.9.

| Role | Description |
| - | - | 
| <pre>certificate-generate_ca</pre> | Role to generate a CA root certificate. |
| <pre>certificate-generate_ca_intermediate</pre> | Role to generate an intermediate CA signed by a CA certificate. |
| <pre>certificate-generate</pre> | Role to generate a SSL certificate for a server or service, signed by a CA (root or intermediate).<br />Can be kept locally or pushed directly on the target hosts |
| <pre>certificate-push-private</pre> | Push a private SSL key and the public certificate on servers |
| <pre>certificate-push-trusted</pre> | Push a public certificate on hosts and add it to their system trusted store and refresh it. |


## Roles for initializing hosts

Some roles to reconfigure part of a host, like with a template or a new server generated from a cloud image.  

| Role | Description |
| - | - | 
| <pre>init-ansible_user</pre> | Used mostly for an interactive playbook, allow to connect to a host using an admin user/password and create the ansible user. |
| <pre>init-server_ansible</pre> | Install the additional system requirements for an Ansible server<br />Mostly an example to build a role to fully reconfigure the system of a new host. |
| <pre>init-server_drive_format</pre> | Role to format one or multiple drives as a LVM device. Complex LVM structures are not supported, each drive will have its own dedicated PV/VG/LV.<br />Given the sensibility of such action, any drive with a hint of an existing partition of any type, even if unusable, will be fully skipped. |
| <pre>init-server_swap</pre> | Manage the system swap. Able to create or increase it using swap files. |


## Role for a monitoring stack

These can install Prometheus + alertmanager, Grafana and deploy multiple agents.  
Personal dashboards are also provided.
| Role | Description |
| - | - | 
| <pre>monitoring-prometheus</pre> | Install Prometheus + Alertmanager using the github archives. Multiple templates are provided, making use of the ansible inventory.<br />A rules and an alerting files are provided, but both are still static under templates/ and must be changed directly. |
| <pre>monitoring-prometheus_agent</pre> |  Install one or multiple prometheus agents as desired using the github archives. Given this role supports multiple agents, it is recommended to activate them one by one the first time. |
| <pre>monitoring-grafana</pre> | Install Grafana using the official deb or rpm package of a specific version (the official repo itself is not activated).<br />Support also the provisioning of a lot of stuff in Grafana v9.x |
| <pre>monitoring-grafana-dashboard</pre> | Grafana dashboard for Prometheus metrics.<br />Not a real role, more of a static companion for provisioning a Grafana instance. |


## Other roles

Various roles.  
They are mostly samples for using the function roles.

| Role | Description |
| - | - | 
| <pre>common</pre> | A role meant to be called each time, generating additional groups using the `os_*` facts and fixing some stuff for Windows hosts. Also list all hosts and each group they are a member of. |
| <pre>dns-bind</pre> | Install and fully configure the Bind dns service. Multiple templates are available, for different zone types, making use of the Ansible inventory.<br />On RHEL-like distribution, the configuration will be heavily altered and moved under /etc/named/ to be more inline with Debian's, thus compatible for both. |
| <pre>ntp-chrony</pre> | Install and configure the ntp server Chrony using the system packages. Both server and client modes are supported. |
| <pre>sys-docker</pre> | Install and configure Docker using the official repo. Also install docker-compose using the binary from github. |
| <pre>sys-packages-update</pre> | Execute the package manager to update the host systems. Multiple modes are supported (security only, full, with reboot, ...).<br />Due to some quirks, apt and dnf/yum are directly used instead of the Ansible modules. |
| <pre>app-vaultwarden</pre> | Install locally Vaultwarden (on-premise password manager) using the docker image. |


---
# Other stuff

## The good stuff

All those roles come with a readme, and are ready to be used.  
Some of them might require to spend some time reading and experimenting with the parameters, but it won't take usually long.


The roles (user_mng and web-*) with the capability to search in the inventory for a specific declaration/suffix make all use of the exact same task, at the start of their tasks/main.yml file.  
Just change the suffix, add a validation task and you can reuse the mechanism in any other role.  
The  mechanism used in the task is functional since Ansible v2.1, if not before.


Some tidbits which could help :  
* a parameter named `project_root` can appear in some roles. It is meant to target a dedicated drive/filesystem for installing any service or application, like under /opt, /server, /app, ..., and separated from the system drive.
* some extra information like the environment, domain, and datacenter are expected, but they will default to "unknown" if not provided. Don't be surprised about them and keep in mind you can fill them later, no need to rush.
* Role's variables have a normalized naming scheme : variables with the `role_` prefix come from vars/, those with `task_` are declared on the fly under tasks/, those starting with `vars_` are usually declared in a task for itself, to simplify a complex declaration.  
  Other starting with the name of the service or module are usually under defaults/ to be used in the inventory.  
* mostly the functions roles, but others too, are split in one or multiple task files, with their execution depending of a condition. This is meant to reduce the output clutter, and speed up the execution.
* some roles, like the functions or the certificates, have a validation task which will raise an error when some parameters are missing or not in the allowed values.


## The bad stuff

Many of those roles took months, sometimes years to handle a lot of real-life situations or compensating for problems they were not meant to handle, but still had to. As such, those doing the heavy lifting are complex, and might not be the best to start with if you don't already known your way with Ansible.  
On the other hand, they will give a good example of different working possibilities when you are stuck on a particular situation.  
The roles with the most various situation are those for the monitoring, starting with the agents. The most difficult are the web-* roles.

In the same manner, while there are many parameters available, not all possible settings are handled by the roles. Thus you might find them lacking or missing for your needs.  
Given how the roles are built, check if declaring a new parameter under defaults/ and inserting it directly in the correct file under templates/ would do the trick.

As with any code, there's always a spaghetti plate hidden somewhere. The function roles are a 'nice' example, but the biggest one will be the inventory. This is the side effect of pushing onto roles to allow more reusability and able to handle multiple situations, the structure of the inventory will grow very fast.  
By the way, don't go too crazy at start with the nginx role.


## The ugly stuff

These roles are provided as-is. They are not available in collections, as I don't have much time keeping up with the maintenance.  
For the same reason, the repository itself will rarely be updated. Maybe from time to time, but this is not the initial objective.  

If you want to take over, please, by all means.  


## Licence 
CC-BY-4.0
