---
title: 'Nginx'
date: 2021-08-25
tags: [ 'nginx', 'web', 'server' ]
---

## Usage

### Check Nginx configuration syntax

Test Nginx configuration

```bash
$ sudo nginx -t
```

Test Nginx configuration and dump output:

```bash
$ sudo nginx -T
```

## Configuration

### Hide Nginx server version

To prevent Nginx from displaying it's version number and OS on error pages and
in HTTP response headers, turn off the `server_tokens` directive in the http
context in the `/etc/nginx/nginx.conf` configuration file.

```bash
[...]
http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;
[...]
```

### Client certificate authentication

#### OpenSSL coniguration

Config file is `/etc/ssl/openssl.cnf`:

```bash
[ ca ]
default_ca = CA_default                 # The name of the CA configuration to be used. 
                                        # can be anything that makes sense to you.
[ CA_default ]
dir = /etc/ssl/ca                       # Directory where everything is kept
certs = $dir/certs                      # Directory where the issued certs are kept
crl_dir = $dir/crl                      # Directory where the issued crl are kept
database = $dir/index.txt               # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several certificates with same subject.
new_certs_dir = $dir/certs              # Default directory for new certs.

certificate = $dir/ca.crt               # The CA certificate
serial = $dir/serial                    # The current serial number
crlnumber = $dir/crlnumber              # The current crl number
                                        # must be commented out to leave a V1 CRL
crl = $dir/crl.pem                      # The current CRL
private_key = $dir/private/ca.key       # The private key
RANDFILE    = $dir/private/.rand        # private random number file

x509_extensions = usr_cert              # The extentions to add to the cert

name_opt = ca_default                   # Subject Name options
cert_opt = ca_default                   # Certificate field options

default_days    = 365                   # how long to certify for
default_crl_days= 30                    # how long before next CRL
default_md    = sha1                    # use public key default MD
preserve    = no                        # keep passed DN ordering

policy = policy_match
```

#### Directory creation:

```bash
$ mkdir -p /etc/ssl/ca/certs/users && \
$ mkdir /etc/ssl/ca/crl && \
$ mkdir /etc/ssl/ca/private
```

#### Database index and CRL number creation:

```bash
$ touch /etc/ssl/ca/index.txt && echo 01 > /etc/ssl/ca/crlnumber
```

#### Certificate Authorithy creation:

```bash
$ openssl genrsa -des3 -out /etc/ssl/ca/private/ca.key 4096
$ openssl req -new -x509 -days 1095 \
    -key /etc/ssl/ca/private/ca.key \
    -out /etc/ssl/ca/certs/ca.crt
$ openssl ca -name CA_default -gencrl \
    -keyfile /etc/ssl/ca/private/ca.key \
    -cert /etc/ssl/ca/certs/ca.crt \
    -out /etc/ssl/ca/private/ca.crl \
    -crldays 1095
```

#### User certificate generation:

```bash
$ openssl genrsa -des3 -out /etc/ssl/ca/certs/users/USERNAME.key 1024
$ openssl req -new -key /etc/ssl/ca/certs/users/USERNAME.key \
    -out /etc/ssl/ca/certs/users/USERNAME.csr
$ openssl x509 -req -days 1095 \
    -in /etc/ssl/ca/certs/users/USERNAME.csr \
    -CA /etc/ssl/ca/certs/ca.crt \
    -CAkey /etc/ssl/ca/private/ca.key \
    -CAserial /etc/ssl/ca/serial \
    -CAcreateserial \
    -out /etc/ssl/ca/certs/users/USERNAME.crt
$ openssl pkcs12 -export -clcerts \
    -in /etc/ssl/ca/certs/users/USERNAME.crt \
    -inkey /etc/ssl/ca/certs/users/USERNAME.key \
    -out /etc/ssl/ca/certs/users/USERNAME.p12
```

#### Nginx configuration:

```bash
$ ssl_client_certificate /etc/ssl/ca/certs/ca.crt;
$ ssl_crl /etc/ssl/ca/private/ca.crl;
$ ssl_verify_client on;
```

#### User revocation:

```bash
$ openssl ca -name CA_Default \
    -revoke /etc/ssl/ca/certs/users/USERNAME.crt \
    -keyfile /etc/ssl/ca/private/ca.key \
    -cert /etc/ssl/ca/certs/ca.crt
$ openssl ca -name CA_Default -gencrl \
    -keyfile /etc/ssl/ca/private/ca.key \
    -cert /etc/ssl/ca/certs/ca.crt \
    -out /etc/ssl/ca/private/ca.crl \
    -crldays 1095
```
