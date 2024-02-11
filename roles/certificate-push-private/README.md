# Ansible role: certificate-push-private

Push private ssl certificates (key and public) under the directory `/etc/ssl/private` on the target servers.  
A subdirectory reusing the certificate name will be created, restricted to root and the `ssl-cert` group.
This group will be created if required, giving read access only on the certificate files.

The role with return the fact `_certificate_push_private_status` with the status `changed` if any certificate was updated.

Please note the private key should at least also secured in a real vault instead of only in the ansible's inventory vault.


## Restrictions and limitations

This role is not meant to generate certificates. For such need, use the `certificate-generate` role instead.  

All certificates to push must be available on the Ansible controller.  
Either in a vault, or as a file in the system.

The role does not update any Java cacert keystore. Check the keytool utility if required.

## Requirements

None.


## Parameters

### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| certificates_private_push | list with the full path of the full data of the certificate files to send. | list[ object ] | [ ] |
| - | certificate name, will be used as the directory name | name: "string" | mandatory |
| - | certificate pem content | pem: "string" | mandatory |
| - | private key content | key: "string" | mandatory |
| - | create an extra file combining both certificate and key.<br />This is required for some server like haproxy. | combined_create: boolean | no |
| - | filter to limit the copy ony to these servers.<br />If empty, all servers will be targeted | target_hosts: [ "string" ] | [ ] |


## Usage examples

The following role's settings are to be used either in a playbook, or set in Ansible's inventory.

```
- role: certificate-push-private
  vars:
    certificates_private_push:
      - name: "sslCert"
        pem: "-----BEGIN CERTIFICATE-----\nMI...M=\n-----END CERTIFICATE-----"
        key: "-----BEGIN PRIVATE KEY-----\nMI...==\n-----END PRIVATE KEY-----"

      - name: "myOtherCert"
        ...
        combined_create: yes
        # target 2 specific hosts and a full inventory group
        target_hosts: [ "host3", "host5" ] + groups['inventory_group2']

      # Other supported constructs for the pem and key parameters:
      # Option 1 : using a yaml scalar - carriage returns will be kept
      - name: "sslCertScalar"
        pem: |
        -----BEGIN CERTIFICATE-----
        MIIFvj...
        ...
        -----END CERTIFICATE-----
        key: |
        -----BEGIN PRIVATE KEY-----
        MI...
        ...==
        -----END PRIVATE KEY-----

      # Option 2 : file content lookup
      # The requested files must be readable by ansible
      - name: "sslCertFile"
        pem: "{{ lookup('file', '/opt/ansible/storage/certs/myCert.pem') }}"
        key: "{{ lookup('file', '/opt/ansible/storage/certs/myCert.key') }}"

      # Option 3: target with a json query a specific cert from a construct in a vault variable (vault_ssl_certs)
      # json_query use ` ` instead of ' ' for the [?name == `target`] as a third type of quote
      - name: "sslCertFromJsonQuery"
        pem: "{{ vault_ssl_certs |json_query('[?name == `myCertOpt3` ].pem') |first }}"
        key: "{{ vault_ssl_certs |json_query('[?name == `myCertOpt3` ].key') |first }}"
        combined_create: yes
```


Any role started after his one can make use of the execution fact with a condition similar to this example :
```
- name: my web server - reloading
  service:
    name: ...
    state: restarted
  when: _certificate_push_private_status is defined and _certificate_push_private_status is changed
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

