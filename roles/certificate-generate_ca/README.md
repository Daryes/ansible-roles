# Ansible role: certificate-generate_ca

create or refresh a CA certificate, used to sign other certificates.

If the CA already exist, the following tests are performed:

- the validity date isn't expired
- the certificate will not expire in the next 7 days
- the common name (CN) is as requested
- the parameter CA:TRUE is present

if all the tests are valid, the existing certificate will also be registered as valid, and skipped.  
Otherwise, it will be recreated, replacing the old one (a local backup will be made).


## Restrictions and limitations

To be able to push the certificate on the hosts, it will be stored as a file on the Ansible controller.  
The files and directories will be restricted and have their owner changed to the user root (mode 600)

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

#### CA public certificate variables

These settings are related to the public certificate usage and expiration.

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| ca_certificate_dir | directory storing the certificate on the server. | "string" | "/etc/ssl/private" |
| ca_certificate_filename_no_ext | Filename for the certificate, without extension. | "string" | "testCA" |
| ca_certificate_validity_duration | number of day the certificate will be valid - format: "+????d"<br />Default is ~20 years, as the 1 year limitation does not apply. | "string" | "+7200d" |
| ca_certificate_usages | List of possible usages<br />Ref: https://www.openssl.org/docs/manmaster/man5/x509v3_config.html => "Key Usage"<br />For a CA, `keyCertSign` is mandatory, `cRLSign` is optional (used to sign cert revocations) | list[ "string" ] | [ "keyCertSign", "cRLSign" ]
| ca_certificate_force_replacement | Set to yes to replace the existing certificate.In some case, a valid certificate could be replaced | boolean | no |

#### CA Private key variables

These settings are related to the private key, encryption and complexity.  

Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| ca_certificate_privatekey_passphrase | private key passphrase - can be empty if really required | "string" | "CHANGE_ME_ASAP"
| ca_certificate_privatekey_type | private key type - carefull to the upper/lower case<br />Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html#parameter-type | "string" | "RSA" |
| ca_certificate_privatekey_size | Key size, in bits | numeric | 4096 |

#### CA certificate CSR variables

These settings are related to the additional certificate informations.

Ref: https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_csr_pipe_module.html

| Parameter | Description | Type | Default value |
| --------- | ----------- | ---- | ------------- |
| ca_certificate_common_name | Certificate common name | "string" | "My Own CA" |
| ca_certificate_email_address | Contact email address | "string" | "someone@wsomewhere.tld" |
| ca_certificate_digest | Certificate digest when signed with the private key | "string" | "sha512" |
| |
| ca_certificate_country_name | Country code, 2 letters | "string" | "FR" |
| ca_certificate_organization_name | Organization name, also known as O<br />Usually the company name, visible on the final certificate | "string" | "company" |
| ca_certificate_organization_unit | Organization unit, also known as OU<br />Usually the company branch, can be omited | "string" | "" |
| ca_certificate_locality_name | Locality name, also known as L<br />Usually the city, can be omited | "string" | "" |
| ca_certificate_locality_state_or_province | State or province name, also known as ST<br />Usually the province of the city, can be omited | "string" | "" |


## Usage examples

The following role's settings are to be used either in a playbook, or set in Ansible's inventory.

```
- role: certificate-generate_ca
  vars:
    ca_certificate_dir: "/etc/ssl/private"
    ca_certificate_filename_no_ext: "ca_file_name"

    ca_certificate_validity_duration: "+7200d"
    ca_certificate_force_replacement: no

    ca_certificate_privatekey_passphrase: "a_passphrase_for_the_private_key - very sensitive"
    ca_certificate_privatekey_type: "RSA"
    ca_certificate_privatekey_size: 4096

    ca_certificate_common_name: "visible name"
    ca_certificate_email_address: "contact@company.tld"
    ca_certificate_country_name: "FR"
    ca_certificate_organization_name: "company"
    ca_certificate_organization_unit: "Unit name"
    ca_certificate_digest: "sha512"

```


# Licence and informations

Provided in the [meta file](meta/main.yml)

