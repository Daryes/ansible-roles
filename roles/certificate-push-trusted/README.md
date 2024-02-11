# Ansible role: certificate-push-trusted

Push CA or domain public certificates, usually self-signed, to the system certificate trust store of the target servers.  
It will allow, using curl or https without any issuer error on a website using said certificate directly or in a chain. 

The java cacert is usually linked to the system rebuild command, and will be updated automatically with the new certificates.

The role with create and return the fact `_certificate_push_trusted_status` with the status `changed` if any certificate was updated.


## Restrictions and limitations

All certificates to push must be available on the Ansible controller.  
Either in a vault, or as a file in the system.

The role does not update the Java cacert keystore directly. It is usually done by a hook from the distribution packages linking the java keystore to the system certificate store.  
If such hook does not exist on your distribution, get a look to the keytool utility.

## Requirements

None.


## Parameters

### Mandatory parameters

None.

### Optional parameters

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| local_certificates_public_push | list with the full path of the public certificate files to send into the servers certs store. | list[ "string" ] | [ ] |
| data_certificates_public_push | List of public certificates data to send into the servers certs store. | list[ "string" ] | [ ] |


## Usage examples

The following role's settings are to be used either in a playbook, or set in Ansible's inventory.

```
- role: certificate-push-trusted
  vars:
    # push certificates from files on the ansible server
    local_certificates_public_push:
      - "/etc/ssl/private/ca/my_own_ca.pem"
      - "/opt/ansible/templates/self_signed_domain.tld.pem"
      - "..."

    # push certificates with their definition stored directly into variables
    data_certificates_public_push:
    - name: "self_signed_domain.tld"
      # use a yaml scalar - carriage returns will be kept
      pem: |
      -----BEGIN CERTIFICATE-----
      MIIFvj...
      ...
      -----END CERTIFICATE-----

    - name: "my_own_ca"
      pem: "{{ vault.certificates.my_own_ca.pem }}"

    - name: ...

```

Any role started after his one can make use of the execution fact with a condition similar to this example : 
```
- name: my web server - reloading
  service:
    name: ...
    state: restarted
  when: _certificate_push_trusted_status is defined and _certificate_push_trusted_status is changed
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

