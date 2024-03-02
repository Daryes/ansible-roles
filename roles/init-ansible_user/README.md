# Ansible role: init-ansible_user

Create the ansible user and give sudoers rights using raw commands.  

It will :
* clear any server ssh key in the ansible controller cache
* connect to the target server using raw command to install python if missing
* connect to the target server using raw command to install sudo if missing
* create the ansible user and group, with sudo rights


## Restrictions and limitations

Both `ansible_user` and `ansible_ssh_pass` must be set in the playbook to allow a connection to the target server with a user+password having sudo rights.

As a suggestion, use a temporary password for the admin user, then with another role, update the user with the definitive password.

Notice : this role must be executed with `become: no`  


## Requirements

None.


## Parameters

### Mandatory parameters

| Parameter | Description | Type |
| --------- | ----------- | ---- |
| ansible_user | User for connecting on the target host.<br />It must already has sudo rights on the host | "string" |
| ansible_ssh_pass | Password in clear text for the connecting user. | "string" | 


### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| ansible_user_shell | Shell for the ansible user | "string" | "/bin/bash" |
| domain_ansible_user | ansible user name | "string" | "ansible" |
| domain_ansible_user_uid | ansible user UID | "string" or numeric | "" |
| domain_ansible_group | ansible group name | "string" | "{{ domain_ansible_user }}" |
| domain_ansible_group_guid | ansible group GUID | "string" or numeric | "{{ domain_ansible_user_uid }}" |
| domain_ansible_ssh_pubkey_file | ansible public key location on the ansible local server | "string" | "~/.ssh/id_rsa.pub" |
| network_dns_dedicated | set to false if you want to add to the ansible controller each remote servers in the `/etc/hosts` file | boolean | true |


## Usage examples

This is the definition of an interactive playbook making use of the init-ansible_user role.  
It will refresh the fact on the local ansible server, then will switch to an interactive mode and ask for the user to connect with to the remote host, and the according password.

The playbook will enforce the the use of the parameter `--limit server1,server2,server3,...`   

```
- name: Playbook environment preparation and validation
  hosts: "{{ ansible_limit | default('localhost') }}"
  gather_facts: false
  # required for using ansible on itself for the first installation - combined with the python_interpreter declaration
  connection: local

  pre_tasks:
    - name: Command line syntax check ...
      fail:
        msg: "ERROR: this playbook must be started with: --limit srv1[,srv2,...]"
      when: ansible_limit is not defined

  tasks:
    - name: Gather ansible master facts
      setup:
      delegate_to: "{{ item }}"
      delegate_facts: true
      loop: "{{ groups['ansible'] }}"
  vars:
    ansible_python_interpreter: "{{ ansible_playbook_python }}"


- name: Prepare the new systems for ansible
  hosts: "{{ ansible_limit | default(omit) }}"
  gather_facts: false
  become: no

  roles:
  - init-ansible_user

  # var prompt is restricted to the current step - also doesn't accept variables as default values
  vars_prompt:
  - name: "ansible_user"
    prompt: |
      This playbook will use the default admin user instead of the ansible user.
      Notice: if sudo isn't installed, the user root must have the same password as this user.

      Please enter the user name to connect to the servers:
    default: "adminserver"
    private: no

  - name: "ansible_ssh_pass"
    prompt: "\nUser password "
    private: yes

  vars:
    ansible_become_pass: "{{ ansible_ssh_pass }}"
    ansible_ssh_common_args: "-o StrictHostKeyChecking=no"
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

