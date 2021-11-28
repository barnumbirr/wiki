---
title: "Let's Encrypt"
date: 2021-08-24
tags: [ "Let's Encrypt", 'TLS', 'certificate' ]
---

## certbot

[certbot](https://certbot.eff.org/) is a free, open source software tool for
automatically using Let’s Encrypt certificates on manually-administrated
websites to enable HTTPS.

### HTTP-01 challenge

Install `certbot`:
```bash
$ apt-get update
$ apt-get install certbot
```

Add the following location to the Nginx HTTP vhost:
```bash
location ^~ /.well-known/acme-challenge/ {
    root /var/www/letsencrypt;
    allow all;
}
```

For Apache, use:
```bash
Alias /.well-known/acme-challenge/ "/var/www/letsencrypt/.well-known/acme-challenge/"
<Directory "/var/www/letsencrypt/">
    AllowOverride None
    Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
    Require method GET POST OPTIONS
</Directory>
```

Request a certificate:
```bash
$ certbot certonly --agree-tos --email admin@example.com --webroot -w /var/www/letsencrypt/ --rsa-key-size 4096 -d your_domain.tld -d your_domain2.tld -d your_domain3.tld
```

Generate a Diffie-Hellman key:
```bash
$ openssl dhparam -out /etc/letsencrypt/live/your_domain.tld/dhparam.pem 4096
```

Add the following to the Nginx HTTPS vhost:
```bash
    ssl_certificate /etc/letsencrypt/live/backup.homelab.lu/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/backup.homelab.lu/privkey.pem;
    ssl_dhparam /etc/letsencrypt/live/backup.homelab.lu/dhparam.pem;
    ssl_session_timeout 5m;
    ssl_prefer_server_ciphers on;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"; # six month
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
```

For Apache:
```bash
    SSLEngine On
    SSLCertificateFile /etc/letsencrypt/live/your_domain.tld/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/your_domain.tld/privkey.pem
    # This is valid only for Apache 2.4.8 or later if using OpenSSL 1.0.2 or later
    SSLOpenSSLConfCmd DHParameters /etc/letsencrypt/live/your_domain.tld/dhparam.pem
    # Requires Apache 2.4.36 & OpenSSL 1.1.1
    SSLProtocol -all +TLSv1.3 +TLSv1.2
    SSLOpenSSLConfCmd Curves X25519:secp521r1:secp384r1:prime256v1
    # Older versions
    # SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
    SSLHonorCipherOrder On
    Header always set Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"
    Header always set X-Frame-Options DENY
    Header always set X-Content-Type-Options nosniff
    # Requires Apache >= 2.4
    SSLCompression off
    SSLUseStapling on
    SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
    # Requires Apache >= 2.4.11
    SSLSessionTickets Off
```

*Make sure to visit https://syslink.pl/cipherlist/ to get an updated SSL cipher list for your webserver.*

Setup autorenewal of all the certificates:
```bash
$ crontab -e
30 3 * * 0 /usr/bin/certbot -q renew --renew-hook "systemctl reload nginx"
```

### DNS-01 challenge

Install `certbot`:
```bash
$ apt-get update
$ apt-get install certbot
```

Install `acme-dns-certbot`:
```bash
$ wget https://github.com/joohoi/acme-dns-certbot-joohoi/raw/master/acme-dns-auth.py
$ chmod +x acme-dns-auth.py
$ nano acme-dns-auth.py
```

Add a 3 to the end of the first line:

```
#!/usr/bin/env python3 #Add a 3 to the end of the first line
```

```bash
$ mv acme-dns-auth.py /etc/letsencrypt/
```

Request a certificate:
```bash
$ certbot certonly --agree-tos --manual --manual-auth-hook /etc/letsencrypt/acme-dns-auth.py --preferred-challenges dns --rsa-key-size 4096 --debug-challenges -d your_domain.tld -d your_domain2.tld -d your_domain3.tld
```

Add `CNAME` record to DNS zone:
```bash
_acme-challenge.your_domain.tld CNAME a15ce5b2-f170-4c91-97bf-09a5764a88f6.auth.acme-dns.io.
```

Generate a Diffie-Hellman key:
```bash
$ openssl dhparam -out /etc/letsencrypt/live/your_domain.tld/dhparam.pem 4096
```

Add the following to the Nginx HTTPS vhost:
```bash
    ssl_certificate /etc/letsencrypt/live/backup.homelab.lu/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/backup.homelab.lu/privkey.pem;
    ssl_dhparam /etc/letsencrypt/live/backup.homelab.lu/dhparam.pem;
    ssl_session_timeout 5m;
    ssl_prefer_server_ciphers on;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"; # six month
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
```

For Apache:
```bash
    SSLEngine On
    SSLCertificateFile /etc/letsencrypt/live/your_domain.tld/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/your_domain.tld/privkey.pem
    # This is valid only for Apache 2.4.8 or later if using OpenSSL 1.0.2 or later
    SSLOpenSSLConfCmd DHParameters /etc/letsencrypt/live/your_domain.tld/dhparam.pem
    # Requires Apache 2.4.36 & OpenSSL 1.1.1
    SSLProtocol -all +TLSv1.3 +TLSv1.2
    SSLOpenSSLConfCmd Curves X25519:secp521r1:secp384r1:prime256v1
    # Older versions
    # SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
    SSLHonorCipherOrder On
    Header always set Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"
    Header always set X-Frame-Options DENY
    Header always set X-Content-Type-Options nosniff
    # Requires Apache >= 2.4
    SSLCompression off
    SSLUseStapling on
    SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
    # Requires Apache >= 2.4.11
    SSLSessionTickets Off
```

*Make sure to visit https://syslink.pl/cipherlist/ to get a list of updated ciphers for your webserver.*

Setup autorenewal of all the certificates:
```bash
$ crontab -e
30 3 * * 0 /usr/bin/certbot -q renew --renew-hook "systemctl reload nginx"
```

!!! hint
    If DNS records get wiped by accident, credentials as well as CNAMEs for all
    domains are stored in `/etc/letsencrypt/acmedns.json`.

## acme.sh

[acme.sh](https://acme.sh/) is a pure Unix shell script implementing ACME client
protocol.

### DNS-01 challenge

```bash
$ cd /tmp
$ git clone https://github.com/acmesh-official/acme.sh.git
$ cd acme.sh
$ ./acme.sh --install --nocron --home /opt/etc/acme.sh --accountemail "user@example.com"
$ nano /opt/etc/acme.sh/dnsapi/dns_ovh.sh

# set your variables
#Application Key
OVH_AK="foo"

#Application Secret
OVH_AS="bar"

#Consumer Key
OVH_CK="foobar"

$ /opt/etc/acme.sh/acme.sh --home /opt/etc/acme.sh --server letsencrypt --issue -d example.com --dns dns_ovh
$ crontab -e
30 3 * * 0 "/opt/etc/acme.sh/acme.sh" --cron --home "/opt/etc/acme.sh/" > /dev/null
```
