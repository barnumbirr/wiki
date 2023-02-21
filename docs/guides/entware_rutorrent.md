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
nano nginx-ssl php8 php8-cli php8-fpm php8-mod-bcmath php8-mod-ctype\
php8-mod-curl php8-mod-filter php8-mod-opcache php8-mod-phar php8-mod-session\
php8-mod-xml procps-ng-pgrep python3 python3-pip  rtorrent-rpcsocat unrar\
unzip wget-ssl
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
## Run rtorrent as a daemon (disables user interface, only XMLRPC control)
system.daemon.set = true

## Control rtorrent via UNIX XMLRPC socket
network.scgi.open_local = (cat, "/opt/var/run/rtorrent.sock")
execute.nothrow = chmod, 770, (cat, "/opt/var/run/rtorrent.sock")

## Listening port for incoming peer traffic, don't randomize
network.port_random.set = no
network.port_range.set = 49879-49879

## Set default data save directory, session file path
directory.default.set = /share/data/torrents
session.path.set = /share/data/.rtorrent/.session

## Disable UDP protocol to trackers, disable DHT/PEX BitTorrent protocols and enable traffic encryption if possible
trackers.use_udp.set = no
dht.mode.set = off
protocol.pex.set = no
protocol.encryption.set = allow_incoming, try_outgoing, enable_retry

## Preferred filename encoding
encoding.add = UTF-8

## Disable hash check on torrents that have finished downloading (intensive process)
pieces.hash.on_completion.set = no

## Minimum and maximum number of peers to connect to per torrent while downloading.
throttle.min_peers.normal.set = 32
throttle.max_peers.normal.set = 64

## Minimum and maximum number of peers to connect to per torrent while seeding. (-1 for same value as above)
throttle.min_peers.seed.set = -1
throttle.max_peers.seed.set = -1

## Maximum number of simultaneous downloads and uploads slots per torrent
throttle.max_downloads.set = 32
throttle.max_uploads.set = 32

## Memory address space to map file chunks.
pieces.memory.max.set = 2048M

## Watch directory, saving a torrent file to this directory will automatically start the download.
schedule2 = watch_directory, 5, 5, load.start=/share/data/.rtorrent/watch/*.torrent
schedule2 = untied_directory, 5, 5, stop_untied=/share/data/.rtorrent/watch/*.torrent

## Stop rtorrent from downloading data when disk space is low.
schedule2 = low_diskspace, 15, 60, ((close_low_diskspace, 50G))

## Save rtorrent session data every 5 minutes (default: 20min)
schedule2 = session_save, 300, 300, ((session.save))

## Initialize ruTorrent plugins
execute2 = {sh,-c,/opt/bin/php-cli /opt/share/www/rutorrent/php/initplugins.php &}

## Logging:
## levels = critical error warn notice info debug
## groups = connection_* dht_* peer_* rpc_* storage_* thread_* tracker_* torrent_
# Log info
# log.open_file = "logfile", (cat,/opt/var/log/rtorrent.log)
# log.add_output = "debug", "logfile"
# log.add_output = "tracker_debug", "logfile"
```

Create rTorrent init file:

!!! Note
    This init file is heavily inspired by Entware's
    `/opt/etc/init.d/rc.func`.


```bash
$ nano /opt/etc/init.d/S85rtorrent
#!/bin/sh

PROCS=rtorrent
ARGS="-D -n -o import=/share/data/.rtorrent/.rtorrent.rc"
DESC=$PROCS
PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

ansi_red="\033[1;31m";
ansi_white="\033[1;37m";
ansi_green="\033[1;32m";
ansi_yellow="\033[1;33m";
ansi_blue="\033[1;34m";
ansi_bell="\007";
ansi_blink="\033[5m";
ansi_std="\033[m";
ansi_rev="\033[7m";
ansi_ul="\033[4m";

ACTION=$1

start() {
    echo -e -n "$ansi_white Starting $DESC... $ansi_std"
    if [ -n "$(pgrep -x -f "$PROC $ARGS")" ]; then
        echo -e "            $ansi_yellow already running. $ansi_std"
        return 0
    fi
    $PROC $ARGS > /dev/null 2>&1 &
    COUNTER=0
    LIMIT=10
    while [ -z "$(pgrep -x -f "$PROC $ARGS")" -a "$COUNTER" -le "$LIMIT" ]; do
        sleep 1;
        COUNTER=$((COUNTER + 1))
    done

    if [ -z "$(pgrep -x -f "$PROC $ARGS")" ]; then
        echo -e "            $ansi_red failed. $ansi_std"
        logger "Failed to start $DESC from $CALLER."
        return 255
    else
        echo -e "            $ansi_green done. $ansi_std"
        logger "Started $DESC from $CALLER."
        return 0
    fi
}

stop() {
    case "$ACTION" in
        stop | restart)
            echo -e -n "$ansi_white Shutting down $PROC... $ansi_std"
            kill "$(pgrep -x -f "$PROC $ARGS")" 2>/dev/null
            COUNTER=0
            LIMIT=10
            while [ -n "$(pgrep -x -f "$PROC $ARGS")" -a "$COUNTER" -le "$LIMIT" ]; do
                sleep 1;
                COUNTER=$((COUNTER + 1))
            done
            rm /opt/var/run/rtorrent.sock
            ;;
        kill)
            echo -e -n "$ansi_white Killing $PROC... $ansi_std"
            kill -9 "$(pgrep -x -f "$PROC $ARGS")" 2>/dev/null
            rm /opt/var/run/rtorrent.sock
            ;;
    esac

    if [ -n "$(pgrep -x -f "$PROC $ARGS")" ]; then
        echo -e "       $ansi_red failed. $ansi_std"
        return 255
    else
        echo -e "       $ansi_green done. $ansi_std"
        return 0
    fi
}

check() {
    echo -e -n "$ansi_white Checking $DESC... "
    if [ -n "$(pgrep -x -f "$PROC $ARGS")" ]; then
        echo -e "            $ansi_green alive. $ansi_std";
        return 0
    else
        echo -e "            $ansi_red dead. $ansi_std";
        return 1
    fi
}

for PROC in $PROCS; do
    case $ACTION in
        start)
            start
            ;;
        stop | kill )
            check && stop
            ;;
        restart)
            check > /dev/null && stop
            start
            ;;
        check)
            check
            ;;
        *)
            echo -e "$ansi_white Usage: $0 (start|stop|restart|check)$ansi_std"
            exit 1
            ;;
    esac
done
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

#### Create ruTorrent 'user' config

We're not creating any ruTorrent users so we'll edit the default `config.php` file.

```bash
$ nano /opt/share/www/rutorrent/conf/config.php
```

Add the following:


```php
<?php
        // configuration parameters

        // for snoopy client
        $httpUserAgent = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36';
        $httpTimeOut = 30;                      // in seconds
        $httpUseGzip = true;
        $httpIP = null;                         // IP string. Or null for any.
        $httpProxy = array
        (
                'use'   => false,
                'proto' => 'http',              // 'http' or 'https'
                'host'  => 'PROXY_HOST_HERE',
                'port'  => 3128
        );

        // for xmlrpc actions
        $rpcTimeOut = 5;                        // in seconds
        $rpcLogCalls = false;
        $rpcLogFaults = true;

        // for php
        $phpUseGzip = true;
        $phpGzipLevel = 2;

        $schedule_rand = 10;                    // rand for schedulers start, +0..X seconds

        $do_diagnostic = true;                  // Diagnose ruTorrent. Recommended to keep enabled, unless otherwise required.
        $al_diagnostic = true;                  // Diagnose auto-loader. Set to "false" to make composer plugins work.

        $log_file = '/opt/var/log/rutorrent/errors.log';          // path to log file (comment or leave blank to disable logging)

        $saveUploadedTorrents = true;           // Save uploaded torrents to profile/torrents directory or not
        $overwriteUploadedTorrents = false;     // Overwrite existing uploaded torrents in profile/torrents directory or make unique name

        $topDirectory = '/share/data/torrents';                    // Upper available directory. Absolute path with trail slash.
        $forbidUserSettings = false;

        $scgi_port = 0;
        $scgi_host = "unix:///opt/var/run/rtorrent.sock";

        // For web->rtorrent link through unix domain socket
        // (scgi_local in rtorrent conf file), change variables
        // above to something like this:
        //
        // $scgi_port = 0;
        // $scgi_host = "unix:///tmp/rpc.socket";

        $XMLRPCMountPoint = "/RPC2";            // DO NOT DELETE THIS LINE!!! DO NOT COMMENT THIS LINE!!!

        $throttleMaxSpeed = 327625*1024;        // DO NOT EDIT THIS LINE!!! DO NOT COMMENT THIS LINE!!!
        // Can't be greater then 327625*1024 due to limitation in libtorrent ResourceManager::set_max_upload_unchoked function.

        $pathToExternals = array(
                "curl"   => '/opt/bin/curl',
                "gzip"   => '/opt/bin/gzip',
                "id"     => '/usr/bin/id',
                "php"    => '/opt/bin/php-cli',
                "pgrep"  => '/opt/bin/pgrep',
                "python" => '/opt/bin/python3',
                "stat"   => '/opt/bin/stat'
        );

        $localHostedMode = true;                // Set to true if rTorrent is hosted on the SAME machine as ruTorrent
        $cachedPluginLoading = true;            // Set to true to enable rapid cached loading of ruTorrent plugins

        $localhosts = array(                    // list of local interfaces
                "127.0.0.1",
                "localhost",
        );

        $profilePath = '../../share';           // Path to user profiles
        $profileMask = 0777;                    // Mask for files and directory creation in user profiles.
                                                // Both Webserver and rtorrent users must have read-write access to it.
                                                // For example, if Webserver and rtorrent users are in the same group then the value may be 0770.

        $tempDirectory = null;                  // Temp directory. Absolute path with trail slash. If null, then autodetect will be used.

        $canUseXSendFile = false;               // If true then use X-Sendfile feature if it exist

        $locale = "UTF8";

        $enableCSRFCheck = false;               // If true then Origin and Referer will be checked
        $enabledOrigins = array();              // List of enabled domains for CSRF check (only hostnames, without protocols, port etc.).
                                                // If empty, then will retrieve domain from HTTP_HOST / HTTP_X_FORWARDED_HOST
```

Configure ruTorrent plugins follows:


```
$ nano /opt/share/www/rutorrent/conf/plugins.ini
[default]
enabled = user-defined
canChangeToolbar = yes
canChangeMenu = yes
canChangeOptions = yes
canChangeTabs = yes
canChangeColumns = yes
canChangeStatusBar = yes
canChangeCategory = yes
canBeShutdowned = yes

[ipad]
enabled = no

[httprpc]
enabled = no

[retrackers]
enabled = no

[rpc]
enabled = no

[rutracker_check]
enabled = no

[geoip]
enabled = no

[geoip2]
enabled = yes

[spectrogram]
enabled = no
```

#### Configure `create` plugin

```bash
$ nano /opt/share/www/rutorrent/plugins/create/conf.php
$useExternal = 'mktorrent';
$pathToCreatetorrent = '/opt/bin/mktorrent';
```

#### Install `cloudscraper` plugin

`cloudscraper` is a dependency of the `_cloudflare` plugin.

```bash
$ pip3 install cloudscraper
```

#### Install `ratiocolor` plugin

```bash
$ cd /opt/share/www/rutorrent/plugins
$ git clone git@github.com:Micdu70/rutorrent-ratiocolor.git ratiocolor
```

#### Install `geoip2` plugin

```bash
$ cd /opt/share/www/rutorrent/plugins
$ git clone git@github.com:Micdu70/geoip2-rutorrent.git geoip2
```

#### Install `nfo` plugin

```bash
$ cd /opt/share/www/rutorrent/plugins
$ git clone git@github.com:phracker/ruTorrent-nfo.git nfo
# Fix for ruTorrent v4.0+
$ sed -i 's/cachedEcho/CachedEcho::send/' nfo/action.php
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
upload_max_filesize = 64M
max_file_uploads = 64
date.timezone = "Europe/Luxembourg"
sys_temp_dir = "/opt/tmp"
```

For PHP-FPM:

```bash
$ nano /opt/etc/php8-fpm.d/www.conf
user = admin
env[PATH] = /opt/bin:/opt/sbin:/bin:/sbin:/usr/bin:/usr/sbin:/usr/bin/X11:/usr/local/sbin:/usr/local/bin
```

#### Allow PHP-FPM to run as root

```bash
$ nano /opt/etc/init.d/S79php8-fpm
ARGS="--daemonize -R --fpm-config /opt/etc/php8-fpm.conf"
```

!!! note
    You'll need to edit the init file to allow `php-fpm` to run as root each
    time the package is updated.

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
    tcp_nodelay off;
    keepalive_timeout 65;
    keepalive_requests 100;
    keepalive_disable msie6;
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
    gzip_min_length 512;
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

    error_page 500 502 503 504 /50x.html;

    location /RPC2 {
        include scgi_params;
        scgi_pass unix:/opt/var/run/rtorrent.sock;
    }

    location = /50x.html {
        root /opt/share/nginx/html;
    }

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location ~ ^/(conf|share)/(.+)$ {
        deny all;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        # Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass unix:/opt/var/run/php8-fpm.sock;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~* \.(jpg|jpeg|gif|css|png|js|map|woff|woff2|ttf|svg|eot)$ {
        expires 30d;
        access_log off;
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
$ wget https://ssl-config.mozilla.org/ffdhe4096.txt -O /opt/etc/acme.sh/rutorrent.homelab.lu/dhparam.pem
```

### Restart services

```bash
$ /opt/etc/init.d/S79php8-fpm restart
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
