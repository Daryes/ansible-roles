# Ansible role: init-server_ansible

Complete an ansible installation by installing the remaining requirements.

The role will also activate log generation of ansible executions to the /var/log/ansible directory (default value)


## Restrictions and limitations

None.

## Requirements

mandatory role :
* _include-pkg_install


The role will install all system and python requirements.  
The full list is available in `vars/main.yml`


## Parameters

### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| domain_ansible_user | System user name for ansible<br />The parameter is the exact same as in the init-ansible_user role | "string" | "ansible" |
| domain_ansible_group | System group name for ansible | "string" | "{{ domain_ansible_user }}" |
| domain_ansible_inventory_group | Inventory group containing the ansible controllers | "string" | "ansible" |
| domain_ansible_log | Log directory used for all ansible executions | "string" | "/var/log/ansible" |
| domain_ansible_log_retention_days | Retention in days for the logs | numeric | 7 |
| domain_ansible_config_file | Full path to the ansible.cfg file.<br />The `ansible_config_file` internal variable from ansible v2.11 will be used first before using the default location | "string" | "/etc/ansible/ansible.cfg" |


## Usage examples

Add the following section to a playbook dedicated to initializing new servers.  
Due to the specific targeting of the 'ansible_group' in the hosts list, it will run only on ansible controllers.  
This group must have been defined beforehand.  

```
- name: Install ansible requirements on controllers
  hosts: "ansible_group"
  remote_user: "ansible"
  gather_facts: true
  become: yes

  roles:
    - init-server_ansible

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

