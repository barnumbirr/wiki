---
title: 'Puppet'
date: 2021-08-27
tags: [ 'puppet', 'hiera', 'r10k' ]
---

## Puppetserver

### Sign client certificate

```bash
$ puppetserver ca sign --certname <fqdn>
```

### List all signed client certificates

```bash
$ puppetserver ca list --all
Requested Certificates:
    <fqdn>       (SHA256)  03:C3:0C:A8:CF:35:7A:AC:C0:EE:48:55:9E:C7:03:8F:3D:09:4C:90:84:38:04:B8:C2:BE:E8:94:A1:01:20:29
Signed Certificates:
    <fqdn>       (SHA256)  EA:4D:98:9B:F6:00:EF:CA:6C:45:50:97:93:CD:1D:C5:02:C8:F7:88:D0:6C:3E:5B:3B:D3:50:69:99:71:58:C0
    <fqdn>       (SHA256)  FF:6F:7A:9F:79:01:30:B8:EF:77:20:36:E7:3A:FA:A3:21:D7:8B:10:93:52:0E:CA:6E:8B:4B:0C:07:55:4A:96  alt names: ["DNS:<fqdn> ", "DNS:puppet", "DNS:puppet.example.com"]
    <fqdn>       (SHA256)  E6:42:CE:F5:BC:58:E1:4C:54:B9:75:D4:E5:5D:D6:C8:08:72:E3:EC:92:0A:EC:0D:FC:39:EA:6E:C9:13:80:BD
    <fqdn>       (SHA256)  27:0E:59:A4:D6:A2:95:A3:3E:27:36:55:66:CE:9B:7F:5C:AE:75:77:91:74:78:AD:50:17:5D:21:19:EE:94:8D
    <fqdn>       (SHA256)  59:67:BC:65:57:76:A8:DA:B7:0D:D4:96:62:1D:62:DE:E0:A5:1E:0D:42:60:8F:CF:F7:57:1D:76:7E:B9:B1:4E
    <fqdn>       (SHA256)  3A:CD:99:F8:9A:A6:9A:E5:50:60:AD:8B:76:42:FB:8A:CF:E7:02:85:EA:62:74:FA:C6:6F:F3:CD:88:CE:3B:B3
```

## Hiera

### Lookup data

```bash
$ hiera -c /etc/puppetlabs/puppet/hiera.yaml bind::zones environment=production ::fqdn=<fqdn> -d
DEBUG: 2021-05-11 02:55:57 +0200: Hiera YAML backend starting
DEBUG: 2021-05-11 02:55:57 +0200: Looking up bind::zones in YAML backend
DEBUG: 2021-05-11 02:55:57 +0200: Looking for data source nodes/<fqdn>
DEBUG: 2021-05-11 02:55:57 +0200: Found bind::zones in nodes/<fqdn>
{"example.com"=>
  {"content"=>["type master", "file \"/etc/bind/zones/example.com.\""]},
 "another_example.com"=>
  {"content"=>["type master", "file \"/etc/bind/zones/another_example.com.\""]}}
```
