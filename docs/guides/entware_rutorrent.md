---
title: 'rTorrent/ruTorrent with Nginx & PHP-FPM on Entware'
date: 2021-10-23
tags: [ 'Entware', rTorrent', 'rutorrent', 'nginx', 'php' ]
---

!!! note
    Some assumptions:  
    - SSH access to device  
    - Entware installed  
    - Neither rTorrent/ruTorrent, nor Nginx nor PHP-FPM previously installed  

### Install required base packages

```bash
$ opkg install bash curl ffmpeg git git-http gzip htop iotop mediainfo mktorrent\
nano nginx-ssl php7 php7-cli php7-fpm php7-mod-ctype php7-mod-curl\
php7-mod-session php7-mod-bcmath php7-mod-phar php7-mod-filter php7-mod-json\
php7-mod-opcache php7-mod-xml procps-ng-pgrep python3 python3-pip rtorrent-rpc\
screen socat unrar unzip wget-ssl
```

### Configure rTorrent

Create necessary directories:

```bash
$ mkdir --parents /share/data/{.rtorrent,torrents}
$ mkdir /share/data/.rtorrent/{.session,watch}
```

Create rTorrent configuration file:

```bash
$ nano /share/data/.rtorrent/.rtorrent.rc
network.scgi.open_port = 127.0.0.1:5000
encoding.add = UTF-8
network.port_range.set = 49879-49879
network.port_random.set = no
pieces.hash.on_completion.set = no
directory.default.set = /share/data/torrents
session.path.set = /share/data/.rtorrent/.session
protocol.encryption.set = allow_incoming, try_outgoing, enable_retry
schedule2 = watch_directory,1,1,load.start=/share/data/.rtorrent/watch/*.torrent
schedule2 = untied_directory,5,5,stop_untied=/share/data/.rtorrent/watch/*.torrent
schedule2 = low_diskspace,1,30,close_low_diskspace=50G
schedule2 = session_save, 240, 300, ((session.save))
trackers.use_udp.set = yes
dht.mode.set = off
protocol.pex.set = no
throttle.min_peers.normal.set = 40
throttle.max_peers.normal.set = 100
throttle.min_peers.seed.set = 10
throttle.max_peers.seed.set = 50
throttle.max_uploads.set = 15
execute2 = {sh,-c,/opt/bin/php /opt/share/www/rutorrent/php/initplugins.php &}

#min_peers = 1
#max_peers = 512
#min_peers_seed = -1
#max_peers_seed = -1
#max_uploads = 512
#download_rate = 0
#upload_rate = 0
#network.max_open_files.set = 1024
#network.http.max_open.set = 512
pieces.memory.max.set = 4096M

# Log info
log.open_file = "rtorrent.log", (cat,/opt/var/log/rtorrent.log)
log.add_output = "debug", "rtorrent.log"
```

Create rTorrent init file:


```bash
$ nano /opt/etc/init.d/S85rtorrent
#!/bin/sh

ENABLED=yes
PROCS=rtorrent
ARGS="-D -n -o import=/share/data/.rtorrent/.rtorrent.rc"
PREARGS="screen -dmS rtorrent"
DESC=$PROCS
PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

. /opt/etc/init.d/rc.func
```

Then run:

```bash
$ chmod +x /opt/etc/init.d/S85rtorrent
```

### Install and configure ruTorrent

```bash
$ cd /opt/share/www/
$ git clone https://github.com/Novik/ruTorrent.git rutorrent
```

#### Create ruTorrent user config

```bash
$ mkdir /opt/share/www/rutorrent/conf/users/default
$ nano /opt/share/www/rutorrent/conf/users/default/config.php
```

Add the following:


```php
<?php

$pathToExternals = array(
    "curl"  => '/opt/bin/curl',
    "stat"  => '/opt/bin/stat',
    "php"    => '/opt/bin/php',
    "pgrep"  => '/opt/bin/pgrep',
    "python" => '/opt/bin/python3'
    );

$topDirectory = '/share/data/torrents';
$scgi_port = 5000;
$scgi_host = '127.0.0.1';
$XMLRPCMountPoint = '/DEFAULT';
```

#### Configure `create` plugin

```bash
$ nano /opt/share/www/rutorrent/plugins/create/conf.php
$useExternal = 'mktorrent';
$pathToCreatetorrent = '/opt/bin/mktorrent';
```

#### Install `cloudscraper` plugin

```bash
$ pip3 install cloudscraper
```

!!! hint
    You might need to "fix" `tracklabels` missing icons
    [as described here](../linux/rutorrent.md#fix_missing_label_icons).


### Configure PHP/PHP-FPM

For PHP:

```bash
$ nano /opt/etc/php.ini
memory_limit = 128M
;doc_root = "/opt/share/www"
upload_max_filesize = 16M
max_file_uploads = 64
date.timezone = Europe/Luxembourg
sys_temp_dir = "/opt/tmp"
```

For PHP-FPM:

```bash
nano /opt/etc/php7-fpm.d/www.conf
user = admin
listen.owner = admin
pm.max_children = 25
pm.start_servers = 4
pm.min_spare_servers = 3
pm.max_spare_servers = 5
env[PATH] = /opt/bin:/opt/sbin:/bin:/sbin:/usr/bin:/usr/sbin:/usr/bin/X11:/usr/local/sbin:/usr/local/bin
```

Don't forget to symlink `php` as for some reason `ruTorrent` won't work without
it:

```bash
$ ln -s /opt/bin/php-cli /opt/bin/php
```

#### Allow PHP-FPM to run as root

```bash
$ nano /opt/etc/init.d/S79php7-fpm
ARGS="--daemonize -R --fpm-config /opt/etc/php7-fpm.conf"
```

### Configure Nginx

```bash
$ nano /opt/etc/nginx/nginx.conf
user admin administrators;
worker_processes auto;
pid /opt/var/run/nginx.pid;
include /opt/etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
    use epoll;
    # multi_accept on;
}

http {

    ##
    # Basic Settings
    ##

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /opt/etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /opt/var/log/nginx/access.log;
    error_log /opt/var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_types
        text/css
        text/javascript
        text/xml
        text/plain
        text/x-component
        application/javascript
        application/x-javascript
        application/json
        application/xml
        application/rss+xml
        application/vnd.ms-fontobject
        font/truetype
        font/opentype
        image/svg+xml;

    ##
    # Virtual Host Configs
    ##

    include /opt/etc/nginx/conf.d/*.conf;
    include /opt/etc/nginx/sites-enabled/*;
}

```

#### Configure Nginx SCGI

```bash
$ nano /opt/etc/nginx/scgi_params
scgi_param  REQUEST_METHOD     $request_method;
scgi_param  REQUEST_URI        $request_uri;
scgi_param  QUERY_STRING       $query_string;
scgi_param  CONTENT_TYPE       $content_type;

scgi_param  DOCUMENT_URI       $document_uri;
scgi_param  DOCUMENT_ROOT      $document_root;
scgi_param  SCGI               1;
scgi_param  SERVER_PROTOCOL    $server_protocol;
scgi_param  REQUEST_SCHEME     $scheme;
scgi_param  HTTPS              $https if_not_empty;

scgi_param  REMOTE_ADDR        $remote_addr;
scgi_param  REMOTE_PORT        $remote_port;
scgi_param  SERVER_PORT        $server_port;
scgi_param  SERVER_NAME        $server_name;
```

#### Create Nginx vhost

```bash
$ nano /opt/etc/nginx/sites-available/rutorrent.example.com.conf
server {
    listen 80;
    server_name rutorrent.example.com;
    return 301 https://$server_name$request_uri;

    access_log /opt/var/log/nginx/example.com/rutorrent/access.log;
    error_log /opt/var/log/nginx/example.com/rutorrent/error.log;
}

server {
    listen 443 ssl http2;
    server_name rutorrent.example.com;

    access_log /opt/var/log/nginx/example.com/rutorrent/access.log;
    error_log /opt/var/log/nginx/example.com/rutorrent/error.log;

    ssl_certificate /opt/etc/acme.sh/rutorrent.example.com/fullchain.cer;
    ssl_certificate_key /opt/etc/acme.sh/rutorrent.example.com/rutorrent.example.com.key;
    ssl_dhparam /opt/etc/acme.sh/rutorrent.example.com/dhparam.pem;
    ssl_session_timeout 5m;
    ssl_prefer_server_ciphers on;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    add_header Strict-Transport-Security "max-age=15768000" always; # six month

    charset utf-8;
    index index.html index.php;
    root /opt/share/www/rutorrent;

    client_max_body_size 64M;

    location ^~ /DEFAULT {
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.html;
        include scgi_params;
        scgi_pass 127.0.0.1:5000;
        #scgi_pass unix:/var/run/helios/rtorrent.sock;
    }

    location ^~ /conf/ {
        deny all;
    }

    location ^~ /share/ {
        deny all;
    }

    location ~* \.(jpg|jpeg|gif|css|png|js|map|woff|woff2|ttf|svg|eot)$ {
        expires 30d;
        access_log off;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        #fastcgi_split_path_info ^(.+\.(?:php|phar))(/.*)$;
        include fastcgi_params;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        #Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass unix:/opt/var/run/php7-fpm.sock;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }
}
```

#### Generate TLS certificates for vhost

Install acme.sh:

```bash
$ cd /tmp
$ git clone https://github.com/acmesh-official/acme.sh.git
$ cd acme.sh/
$ ./acme.sh --install --nocron --home /opt/etc/acme.sh --accountemail "user@example.com"
```

Configure your DNS provider API keys:

```bash
$ nano /opt/etc/acme.sh/dnsapi/dns_ovh.sh
#Application Key
OVH_AK="foo"

#Application Secret
OVH_AS="bar"

#Consumer Key
OVH_CK="foobar"
```

Generate TLS certificate:

```bash
$ /opt/etc/acme.sh/acme.sh --home /opt/etc/acme.sh --server letsencrypt --issue -d example.com --dns dns_ovh
```

Setup a cronjob to automatically renew your certificate:

```bash
$ crontab -e
30 3 * * 0 "/opt/etc/acme.sh/acme.sh" --cron --home "/opt/etc/acme.sh" > /dev/null
```

Setup a Diffie-Helmann key as follows:

```bash
wget https://ssl-config.mozilla.org/ffdhe4096.txt -O /opt/etc/acme.sh/rutorrent.homelab.lu/dhparam.pem
```

### Restart services

```bash
$ /opt/etc/init.d/S79php7-fpm restart
$ /opt/etc/init.d/S80nginx restart
$ /opt/etc/init.d/S85rtorrent start
```

!!! hint
    Nginx might error out with the following message:

    ```
    nginx: [error] open() "/opt/var/run/nginx.pid" failed (2: No such file or directory)
    ```

    To fix it, simply run the following command:

    ```
    $ nginx -c /opt/etc/nginx/nginx.conf
    ```

You should now be all set! :tada:
