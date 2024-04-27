# Ansible role: certificate-generate

Create or refresh a SSL certificate, used by a webserver (usually)

If the certificates already exist, the following tests are performed:

- the validity date isn't expired
- the certificate will not expire in the next 7 days
- the common name (CN) is as requested
- the alt names are as requested

if all the tests are valid, the existing certificate will also be registered as valid, and skipped.  
Otherwise, it will be recreated, replacing the old one.

Notice: the certificate is created locally on the ansible server.  
When pushed, the same copy is used for all the targeted servers.  
The local cache can be cleared if requested.


## Restrictions and limitations

To be able to create or renew the certificates, the Ansible controller must have access to a CA certificate (root or intermediate) to be able to sign the new ssl certificates.  

As this role is built around the ansible crypto collection, all restrictions also apply here.  
References : 
- Ansible : https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html#modules
- Openssl : https://docs.ansible.com/ansible/latest/collections/community/crypto/x509_certificate_module.html


Also, give much attention to the character case, many settings use lower and / or upper case in some parameter values, each of them being important. This could lead to a refused certificate, while apparently valid.


## Requirements

Ansible collections :
* community.crypto

Notice: the python cryptography library must be installed server-wide


## Parameters

### Mandatory parameters

None.


### Optional parameters

#### SSL certificate public variables

These settings are related to the public certificate usage and expiration.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| server_certificate_dir | root directory for storing the certificates on the server | "string" | "/etc/ssl/private" |
| server_certificate_owner | system user that will be given ownership of the certificates.<br />Usually, root | "string" | "root" |
| server_certificate_group | system group getting ownership.<br />If empty, it will reuse `server_certificate_owner` value | "string" | "root" |
| server_certificate_validity_duration | Number of day the certificate will be valid - format: "+????d"<br />Currently, a validity exceeding 1 year might not be accepted | "string" | "+365d" |
| server_certificate_subject_name | Certificate subject, usually the requested domain name without wildcards or the server name.<br />It will also be used for the filenames | "string" | "subject.example.tld" |
| server_certificate_alt_name | Subject Alt Name, or SAN. This is the multiples names (with wildcards) the certificate will answer to.<br />The "DNS:" prefix must be present in front of each entry<br />Example:<br />`- "DNS:{{ server_certificate_subject_name }}"`<br />`- "DNS:*.{{ server_certificate_subject_name }}"`<br />`- "DNS:subject-alt.example.tld"`<br />`- "DNS:*.subject-alt.example.tld"` | list[ "string" ] | [ ] |
| server_certificate_chain_add_ca_public_cert | Insert the local_ca public certificate in the server public certificate.<br />Usually required if the CA is an intermediate CA | boolean | no |
| server_certificate_force_replacement | Force the recreation of the certificate even if it is still valid | boolean | no |
| | | | |
| server_certificate_delete_certs_in_ansible_cache | Set to yes to purge the certificates in the ansible cache when finished.<br />Any certificate renewal will also get a new private key | boolean | no |
| ansible_cache_root_dir | Full path for the cache directory used to store the certificates on the ansible controller.<br />It will be created if missing. | "string" | "/opt/ansible/cache" |


Each certificate files and private key will be stored on the servers under : `< server_certificate_dir >/< subject name >/`  
So be carefull with the subject name to avoid duplicate, unless to replace an unused one.


#### SSL private key variables

These settings are related to the private key, encryption and complexity.

Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| server_certificate_privatekey_passphrase | private key passphrase - can be empty if required.<br />If set, the passphrase will be requested by the webserver at each launch | "string" | "" |
| server_certificate_privatekey_type | Private key type - carefull to the upper/lower case<br />Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html#parameter-type | "string" | "RSA" |
| server_certificate_privatekey_size | Key size, in bits | numeric | 4096 |


#### SSL certificate CSR variables

These settings are related to the public certificate informations.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| server_certificate_common_name | Certificate common name | "string" | "{{ server_certificate_subject_name }}" |
| server_certificate_email_address | Contact email address | "string" | "someone@wsomewhere.tld" |
| server_certificate_country_name | Country code, 2 letters | "string" | "FR" |
| server_certificate_organization_name | Organization name<br />It will be listed on the certificate | "string" | "company" |
| server_certificate_organization_unit | Organization unit | "string" | "branch" |
| server_certificate_digest | Certificate digest when signed with the private key | "string" | "sha512" |


#### CA certificate variables

These settings are related to the CA public and private key used to sign the ssl certificates.  
If the type is set to "file", the CA is expected to be located only on the ansible server.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| local_ca_type | Source type for the CA cert.<br />If type=`variable`, use the "*_content" parameters.<br />If type=`file`, use the "*_path" parameters. | "file" or "variable" | "file" |
| local_ca_privatekey_passphrase | Passphrase for the CA private key.<br />The value can be passed from a vault. | "string" | "" |
| local_ca_path | Full path to the CA public certificate (pem or crt).<br />Ex:  "/path/on/ansible/server/to/ca.crt" | "string" | "" |
| local_ca_privatekey_path | Full path to the CA private key<br />Ex: "/path/on/ansible/server/to/ca.key" | "string" | "" |
| local_ca_content | CA public certificate content | "string" | "" |
| local_ca_privatekey_content | Ca private key content | "string" | "" |


## Usage examples

The following role's settings are to be used either in a playbook, or set in Ansible's inventory.

```
- role: certificate-generate
  vars:
    server_certificate_subject_name: "subject.example.tld"

    # the multiple dns name the certificate will support. 'DNS:' must be present, wildcards are supported
    server_certificate_alt_name: [ "DNS:sub.domain.tld", "DNS:*.sub.domain.tld" ]

    server_certificate_validity_duration: "+365d"
    server_certificate_force_replacement: no

    # Server certificate CSR variables
    server_certificate_common_name: "{{ server_certificate_subject_name }}"
    server_certificate_email_address: "contact.mail@sub.domain.tld"
    server_certificate_country_name: "FR"
    server_certificate_organization_name: "company or organization name"
    server_certificate_organization_unit: "a unit name"
    server_certificate_digest: "sha512"

    # Server variables for the private key
    # no passphrase on purpose
    server_certificate_privatekey_passphrase: ""
    server_certificate_privatekey_type: "RSA"
    server_certificate_privatekey_size: 4096

    # the CA on the ansible controller to sign the new ssl certificate with
    local_ca_path: "/path/on/ansible/server/to/ca.crt"
    local_ca_privatekey_path: "/path/on/ansible/server/to/ca.key"
    local_ca_privatekey_passphrase: "CA passphrase - very sensitive"
```


For using a CA stored in a variable, or a vault :
```
# Vault definition in the inventory
vault:
  certificate:
    my_ca:
      # using a yaml scalar - carriage returns will be kept
      pem: |
        -----BEGIN CERTIFICATE-----
        MIIFvj...
        ...
        -----END CERTIFICATE-----
      key: |
        -----BEGIN PRIVATE KEY-----
        MI...
        ...
        -----END PRIVATE KEY-----
      pass: "CA passphrase - very sensitive"


# Playbook definition
- role: certificate-generate
  vars:
    (...)
    local_ca_type: "variable"
    local_ca_content: "{{ vault.certificate.my_ca.pem }}"
    local_ca_privatekey_content: "{{ vault.certificate.my_ca.key }}"
    local_ca_privatekey_passphrase: "{{ vault.certificate.my_ca.pass }}"
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

