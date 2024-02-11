# Ansible role: certificate-generate_ca_intermediate

Create or refresh an intermediate CA certificate, used to sign other certificates.

If the intermediate CA already exist, the following tests are performed:

- the validity date isn't expired
- the certificate will not expire in the next 7 days
- the common name (CN) is as requested
- the parameter CA:TRUE is present

if all the tests are valid, the existing certificate will also be registered as valid, and skipped.
Otherwise, it will be recreated, replacing the old one (a local backup for the key will be made).


## Restrictions and limitations

To be able to create or renew the certificates, the Ansible controller must have access to a CA certificate (root or intermediate) to be able to sign the new ssl certificates.


As this role is built around the ansible crypto collection, all restrictions also apply here.  
Meaning most of the role is designed to be ran on the ansible controller.  
While it is possible to have it ran targeting a remote server, an error might occur about the python cryptography library not detected, which must be installed using pip on said remove server.
References :  
- Ansible : https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html#modules
- Openssl : https://docs.ansible.com/ansible/latest/collections/community/crypto/x509_certificate_module.html


Also, give much attention to the character case, many settings use lower and / or upper case in some parameter values, each of them being important.  
This could lead to a refused or recreated certificate, while apparently valid.


## Requirements

Ansible collections :
* community.crypto

Notice: the python cryptography library must be installed server-wide


## Parameters

### Mandatory parameters

None.

### Optional parameters


#### Intermediate CA public certificate variables

These settings are related to the public certificate usage and expiration.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| cainter_certificate_dir | directory storing the certificate on the server. | "string" | "/etc/ssl/private" |
| cainter_certificate_filename_no_ext | Filename for the certificate, without extension. | "string" | "testIntermediateCA" |
| cainter_certificate_validity_duration | number of day the certificate will be valid - format: "+????d"<br />Default is ~2 years. | "string" | "+720d" |
| cainter_certificate_usages | List of possible usages<br />Ref: https://www.openssl.org/docs/manmaster/man5/x509v3_config.html => "Key Usage"<br />For a CA or intermediate CA, `keyCertSign` is mandatory, `cRLSign` is optional (used to sign cert revocations) | list[ "string" ] | [ "keyCertSign", "cRLSign" ]
| cainter_certificate_force_replacement | Set to yes to replace the existing certificate. In some case, a valid certificate could be replaced | boolean | no |
| cainter_certificate_pathlen | Maximum number of CAs (all types) that can appear below this one in a chain.<br />CA with a pathlen of zero  can only be used to sign end user certificates and no further CAs | numeric | 0 |


#### Intermediate CA Private key variables

These settings are related to the private key, encryption and complexity.  
Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| cainter_certificate_privatekey_passphrase | private key passphrase - can be empty if really required | "string" | "CHANGE_ME_ASAP"
| cainter_certificate_privatekey_type | private key type - carefull to the upper/lower case<br />Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html#parameter-type | "string" | "RSA" |
| cainter_certificate_privatekey_size | Key size, in bits | numeric | 4096 |


#### Intermediate CA certificate CSR variables

These settings are related to the additional certificate informations.  
Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_csr_pipe_module.html

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| cainter_certificate_common_name | Certificate common name | "string" | "My Own CA" |
| cainter_certificate_email_address | Contact email address | "string" | "someone@wsomewhere.tld" |
| cainter_certificate_country_name: | Country code, 2 letters | "string" | "FR" |
| cainter_certificate_organization_name | Organization name<br />It will be listed on the certificate | "string" | "company" |
| cainter_certificate_organization_unit | Organization unit | "string" | "branch" |
| cainter_certificate_digest | Certificate digest when signed with the private key | "string" | "sha512" |


#### CA certificate variables

These settings are related to the CA public and private key used to sign the intermediate CA.  
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
- role: certificate-generate_ca_intermediate 
  vars:
    cainter_certificate_dir: "/etc/ssl/private"
    cainter_certificate_filename_no_ext: "ca_file_name"

    cainter_certificate_validity_duration: "+370d"
    cainter_certificate_force_replacement: no

    cainter_certificate_privatekey_passphrase: "a_passphrase_for_the_private_key - very sensitive"
    cainter_certificate_privatekey_type: "RSA"
    cainter_certificate_privatekey_size: 4096

    cainter_certificate_common_name: "visible name"
    cainter_certificate_email_address: "contact@company.tld"
    cainter_certificate_country_name: "FR"
    cainter_certificate_organization_name: "company"
    cainter_certificate_organization_unit: "Unit name"
    cainter_certificate_digest: "sha512"

    # the CA on the ansible controller to sign the new ssl certificate with
    local_ca_type: "variable"
    local_ca_path: "/path/on/ansible/server/to/ca.crt"
    local_ca_privatekey_path: "/path/on/ansible/server/to/ca.key"
    local_ca_privatekey_passphrase: "CA passphrase - very sensitive"
```


# Licence and informations

Provided in the [meta file](meta/main.yml)

