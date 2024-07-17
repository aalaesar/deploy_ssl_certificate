Deploy SSL certificate
=========

A simple role to deploy or replace ONE ssl certificate in the PEM format to a server in some folder location. It can reload/restart dependant services.

Requirements
------------

community.crypto >= 2.10

Role Variables
--------------

### Providing cert and pkey data

You can either source content from files available on the ansible host or as a string passed directly to the variable.
Set the value of `deployssl_pem_type` to either `content` of `file`,
then set both `deployssl_pem_cert_src` and `deployssl_pem_pkey_src` accordingly.

__PLEASE NOTE:__
In case you provide multiple PEM certificates to `deployssl_pem_cert_src`,
it is assumed that the first PEM cert contain the _main_ subject and the next ones are the _full pem chain_.

If you want to provide the _pem chain_ from a separate source,
you can provide it with `deployssl_pem_chain_src`.
This variable follow the same content source logic as `deployssl_pem_cert_src`.

In case `deployssl_pem_chain_src` is provided AND `deployssl_pem_cert_src` contains a pem chain,
the pem chain content of `deployssl_pem_cert_src` __will take over__.

### certificat installation

You need to specify the full path of the cert and pkey
where the pem certs files should be put on the target host.

_It is __strongly__ recommended to use one simple extension at the end of your file name like `.pem` or `.crt` otherwise it way create unexpected behavior in some filenames generation._

```yaml
deployssl_pem_cert_trgt: /etc/ssl/certs/cert.pem
deployssl_pem_pkey_trgt: /etc/ssl/private/pkey.pem
```

The following 2 booleans change how the role manage the pem chain along the main cert
```yaml
deployssl_split_fullchain: True
deployssl_create_fullchain: False
```

If you set `deployssl_split_fullchain` as __True__,
the target pem file will contains only the main cert and a file with the `_chain` suffix will be created __only__ if a pem chain was provided.

If you set `deployssl_create_fullchain` as __True__,
a new file with the `_fullchain` suffix will be created __only__ if `deployssl_pem_cert_src` contains __1__ cert __and__ a pem chain was provided.
This `_fullchain` file will contains all the certs provided, while the target file will only contain the main cert.

_NOTE: both `chain` and `fullchain` pem files will go in the __same folder__ as the cert._

### Updating the aplications using the cert

The role can trigger some services reload or restart if it changed any file.
Specify the service(s) name(s) in the list depending on the action to take.
```yaml
deployssl_reload_svc: []
deployssl_restart_svc: []
```

### Changing the served subjects

If the target pem file has to be replaced,
the role will check that the new cert cover at least the same list of served subjects.

In case you want to skip this check, set `deployssl_force_deploy` to `True`




Example Playbook
----------------

Provides the cert directly in value content,
install in /etc/apache2/ssl
and create the full pem chain file.
(the role will create `/etc/apache2/ssl/mydomain.com_fullchain.pem`)

```yaml
- hosts: servers
  roles:
      - role: aalaesar.deploy-ssl-certificate
        deployssl_pem_type: content
        deployssl_pem_cert_src: |
          -----BEGIN CERTIFICATE-----
          [...]
          -----END CERTIFICATE-----
        deployssl_pem_pkey_src: |
          -----BEGIN RSA PRIVATE KEY-----
          [...]
          -----END RSA PRIVATE KEY-----
        deployssl_pem_chain_src: |
          -----BEGIN CERTIFICATE-----
          [...]
          -----END CERTIFICATE-----
        deployssl_pem_cert_trgt: /etc/apache2/ssl/mydomain.com.pem
        deployssl_pem_pkey_trgt: /etc/apache2/ssl/mydomain.com.pkey.pem
        deployssl_create_fullchain: True
        deployssl_reload_svc:
          - apache2
```

License
-------

BSD
