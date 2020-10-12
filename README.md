Ansible Role: certificate-deploy
================================

This role creates or regenerates keys and cerificates locally, signs the
certificates locally using LetsEncrypt's `dns-01` authorization and deploys them
to the destination servers.


Requirements
------------

* openssl
* tinydns ([djbdns](http://cr.yp.to/djbdns.html))


Role Variables
--------------

| Variable           | Default value                                             | Description                                |
| ------------------ | --------------------------------------------------------- | ------------------------------------------ |
| `brl_crt_src`      | `/home/luser/letsencrypt/certs`                           | The local directory for keys and certs     |
| `brl_crt_dst`      | `/etc/pki/tls/certs`                                      | The local directory for keys and certs     |
| `brl_key_dst`      | `/etc/pki/tls/private`                                    | The local directory for keys and certs     |
| `brl_le_account`   | `/home/luser/letsencrypt/account.pem`                     | LetsEncrypt account key in PEM format      |
| `brl_le_email`     | `foo@bar.com`                                             | LetsEncrypt account email                  |
| `brl_le_directory` | `https://acme-staging-v02.api.letsencrypt.org/directory`  | The ACME directory.                        |
| `brl_acme_version` | `2`                                                       | The ACME protocol version                  |
| `brl_le_intca`     | `cert.stg-int-x1.letsencrypt.org.crt`                     | The name of the intermediate certificate.  |
| `brl_crt_maxage`   | `604800`                                                  | Max validity (sec) of cert before renewal  |
| `brl_key_maxage`   | `31556736`                                                | Maximum age of the key before regeneration |
| `brl_bitsize`      | `4096`                                                    | RSA key bit size                           |
| `brl_dns_dir`      | `/home/luser/dns`                                         | Tinydns directory                          |
| `brl_dns_file`     | `data`                                                    | Tinydns source file                        |
| `brl_dns_head`     | `'_acme-challenge.`                                       | Tinydns TXT RR prefix                      |
| `brl_dns_tail`     | `:0::`                                                    | Tinydns TXT RR postfix                     |

These variables are defined in `default/main.yml`. Also see the comments in that
file. Edit these defaults to reflect your own environment.

If the certificate still has a validity that is more than `brl_crt_maxage`
seconds (default is 1 week), regenerating the certificate is skipped. You can
force the regeneration of a specific certificate with an "extra-vars"
command-line definition like:
```
ansible-playbook --extra-vars force_replacement=fully.qualified.domain ...
```


Host Variable Format
--------------------
The array `brl_certs` holds the Common Name and Subject Alternative Names for
the certificates that should be generated and deployed to the host. Both the
`fqdn` variable and `san` array elements should be fully qualified domain names.
`san` is optional, all other dictionary keys are mandatory. In the state is
present the certificate is generated (if needed) and deployed. If the state is
absent, the key and certificate are deleted both locally as well as on the
remote host.

```
brl_certs:
  - fqdn: <fqdn>
    engine: apache|nginx|postfix
    state: present|absent
    san:
      - altfqdn1
      - ...
  - ...
```


Platform
--------

This role has only been tested on Red Hat derivates but it is likely to work on
Debian derivates as well. For that you will likely have to change the
destination directories for keys and certificates to `/etc/ssl/private` and
`/etc/ssl/certs` respectively and change the apache handler to reload "apache2"
i.o. "httpd".


DNS authorization
-----------------

The dns authorization supposes a tinydns installation. It is likely not hard to
rewrite the tasks to modify a Bind installation, but keep a little time in mind
for slave servers to update their zones. It also supposes that compiling the
tinydns data file and `scp`-ing the files to the DNS servers is done via a `make`
command in the tinydns directory.


SELinux caveat
--------------

Modifying the DNS source file happens on the local host. If you run ansible from
a virtualenv and you have SELinux in permissive or enforcing mode, you can get
an error that the python selinux bindings are missing. This is because the
python that executes the file modification is the version in the virtualenv. To
overcome this you can add a localhost entry to your inventory with the content
```
localhost ansible_python_interpreter=/usr/bin/python
```


Certificate chains
------------------

This role deploys only two files: a file containing the key (named `"fqdn".key`)
and a file containing both the certificate and LetsEncrypt intermediate CA
certificate (named `"fqdn".chain.crt`). This means that if you use Apache you
don't need to specify the `SSLCACertificateFile` directive. Apache works fine
with a "chained" certificate file while Nginx requires it.


Dependencies
------------

None.


Example Playbook
----------------

```
- hosts: all
  become: yes
  become_method: sudo
  roles:
    - role: certificate-deploy
```



License
-------

GPLv3


Author Information
------------------

Rob Wolfram <propdf@hamal.nl>
