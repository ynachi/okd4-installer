# dns - Ansible role

DNS role creates dns for a public domain on cloudflare.
It also setup SSL certificates using Letsencrypt.

## How to use

To use it, it has be listed in roles from the playbook

```yaml
---
- name: Setup dns record and ssl certificates
  hosts: deployment
  gather_facts: false
  tasks:
    - name: Import dns role
      import_role:
        name: dns
      tags:
       - dns
       - ssl
       - letsencrypt
```

## Mandatory variables

# @TODO: check for redundancy

* pd_public_ip:  public IP for the public domain
* pd_provider: public domain provider, only cloudflare supported for now
* pd_cloudflare_zone: public dns zone
* le_public_domain: ?
* le_cloudflare_zone: public dns zone
* pd_public_domain: public dns domain name
* le_certificates_dir:  where the certificate files will be stored
* le_letsencrypt_account_email: user letsencrypt email address
* le_acme_directory: acme directory (typically prod or staging)
* le_dns_provider: ??? To review

