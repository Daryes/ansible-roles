# Ansible role: _include-serial_over_hostgroups

Objective of this role is to alleviate one of Ansible's limitation which does not allow to combine the `serial` keyword and groups of multiple hosts.  
Both the `serial` playbook setting, and the `throttle` task parameter run only sequentially, without any possibility to separate or regroup the targets hosts.  

This role will return a variable containing the given hosts grouped as desired in multiple sets.  
Each set will contain one combination of hosts, and will continue until all hosts are in a group.  
The number of hosts per set is related to the number of patterns requested :  
* with 2 patterns, each set will have 2 hosts
* with 3 patterns, each set will have 3 hosts
* ...

Having a discrepancy with the number of hosts between the patterns will only result in less hosts in the last sets. They will still be grouped by the patterns, without any of them lost.

The role itself is only half of the solution, the other part is using the role returned variable combined with :
* `include_role` (or task)
* a loop over `"{{ _serial_over_hostgroups }}"` which is returned by the role
* this conditional : `when: inventory_hostname in item.hosts`

Using this conditionnal in a loop will restrict the action (the include_*) to apply only to the hosts in the current set of the loop.

Check the usage examples for a working sample.


This role can be called either as a standalone role, or as a function from inside another role.


## Restrictions and limitations

The patterns used are the same as [the playbook pattern parameter](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html).  
The usual limitations apply here.  
One common caveat is the use of `,` as a separator instead of `:` . Both are accepted, but in the same pattern, it is either one or the other. Having them together will raise an error.

Using patterns for the hosts is possible due to [the inventory_hostnames lookup](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/inventory_hostnames_lookup.html).  
Any change on this lookup will affect the role.  
As such, the use of regex in a pattern might not work as the support seems limited by the lookup.  


The task making use of the set of hosts will generate once on the include task itself a long list of multiple `skipped` lines.  
This is an Ansible limitation, as to have the hosts running in groups, the conditional will create a large matrix opposing each host to each set of hosts.  
Still, this will occur only one time.


To alleviate having missing hosts in the generated list, it is recommended to have `fact_caching` set to a persistent type instead of memory in the `ansible.cfg` file


## Requirements

None.


## Parameters


### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| serial_over_hostgroups_patterns | List of any [host patterns](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/inventory_hostnames_lookup.html) which can be servers, groups or combinations<br />Example:<br /> - "mygroup:&linux"<br />- "server1,server2,server5"<br />- "webserver*:!atlanta" | list[ "string" ] | [ ] |
| serial_over_hostgroups_exclude_servers | List of servers to exclude, patterns are not supported here.<br />Example:<br />- "server1"<br />- "server2"<br />- "{{ groups['mygroup'] }}" | list[ "string" ] | [ ] |


Notice: hosts will usually be sorted in the process.


### Output parameters

The variable `_serial_over_hostgroups` will be returned as a fact, valid only for the current execution.  
It will contains dictionnaries with the following keys : { set: index of the set, size: number of host in the set, hosts: [ host1, host2, ...] }

Example:
Assuming the first group has 2 hosts (1 & 2), the second group has 4 hosts (A-D), and the last group with 3 hosts (x-z).
```
_serial_over_hostgroups:
  - { set: 1, size: 3, hosts: [ 'host1', 'hostA', 'hostx' ] }
  - { set: 2, size: 3, hosts: [ 'host2', 'hostB', 'hosty' ] }
  - { set: 3, size: 2, hosts: [ 'hostC', 'hostz' ] }
  - { set: 4, size: 1, hosts: [ 'hostD' ] }
```


## Usage examples

Having 3 hypervisors, multiples VMs running, balanced between them.  
A `maintenance` role must be executed on each linux VM, but only on a single VM per hypervisor at the same time. One VM simultaneously per hypervisor is fine.

Inventory groups already exist for each hypervisor, listing their own VMs.  
A linux group also exists, containing all the linux servers.  
Note : the module [group_by](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group_by_module.html) can help to generate such group dynamically.

For this situation, using the parameter `serial:` would be the first solution, but will only work if set to 1 due to the desired limitation.  
While valid, it will take as much time as the total number of VM.  


Using this role will greatly reduce the execution duration as it will occur on 3 VM each time, one per hypervisor.


Complete playbook example :
```
- name: Server maintenance
  # Dynamic inventory or groups might require 'all' to build a valid list
  # The role use run_once, and will execute only one time.
  hosts: all
  remote_user: "ansible"
  become: yes

  # In a playbook, the order is : pre_tasks, roles, then tasks
  # this role can be used directly as a role or included by another role

  roles:
    - name: _include-serial_over_hostgroups
      vars:
        serial_over_hostgroups_patterns:
          - "virt_kvm_host_a:&linux"
          - "virt_kvm_host_b:&linux"
          - "virt_kvm_host_c:&linux"
        serial_over_hostgroups_exclude_servers: []

  tasks:
    # Let's see the resulting sets
    - debug:
        msg: "{{ _serial_over_hostgroups }}"
      run_once: yes

    # Run the maintenance role
    # The conditional will have the useful effect to generate one include per set, 
    # restricted to the hosts from the current set.
    # One caveat : a wall of skipped line. But at least just one time on the include itself.
    - include_role: name=maintenance
      with_items: "{{ _serial_over_hostgroups }}"
      when: inventory_hostname in item.hosts

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

