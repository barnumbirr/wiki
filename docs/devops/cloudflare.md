---
title: 'Cloudflare'
date: 2021-10-09
tags: [ 'Cloudflare', 'CDN', 'DNS' ]
---

## Debugging

### Self-inflicted DNS cache poisoning

Cloudflare suddenly and randomly starts returning the following when accessing
`support.example.com`:

```
Please stand by, while we are checking your browser...

RayID: 699eaa277be49730
IP: 162.159.90.246
```

A quick DNS looking returns:

```
$ dig support.example.com

; <<>> DiG 9.16.15-Debian <<>> support.example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21805
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;support.example.com.                IN      A

;; ANSWER SECTION:
support.example.com. 92      IN      A       104.25.11.76
support.example.com. 92      IN      A       104.25.14.76
support.example.com. 92      IN      A       172.69.12.65

;; Query time: 0 msec
;; SERVER: 10.0.1.1#53(10.0.1.1)
;; WHEN: Wed Oct 06 15:02:00 CEST 2021
;; MSG SIZE  rcvd: 88
```

From our Cloudflare DNS zone we know that `support.example.com` is a CNAME entry
for another CNAME, `example.zendesk.com`, the DNS lookup of which is as follows:

```
$ dig example.zendesk.com

; <<>> DiG 9.16.15-Debian <<>> example.zendesk.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1008
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 4, ADDITIONAL: 0

;; QUESTION SECTION:
;example.zendesk.com.                IN      A

;; ANSWER SECTION:
example.zendesk.com. 79      IN      A       104.16.57.111
example.zendesk.com. 79      IN      A       104.16.55.111

;; AUTHORITY SECTION:
zendesk.com.            669     IN      NS      ns-2011.awsdns-59.co.uk.
zendesk.com.            669     IN      NS      ns-709.awsdns-24.net.
zendesk.com.            669     IN      NS      ns-1219.awsdns-24.org.
zendesk.com.            669     IN      NS      ns-128.awsdns-16.com.

;; Query time: 0 msec
;; SERVER: 10.0.1.1#53(10.0.1.1)
;; WHEN: Wed Oct 06 15:02:03 CEST 2021
;; MSG SIZE  rcvd: 212
```

There is a clear IP mismatch between the upstream CNAME and the records in our
zone. **This is due to the Cloudflare DNS entry being proxied.**
Switching the entry to `DNS only` fixes the issue immediately:

```
$ dig support.example.com

; <<>> DiG 9.16.15-Debian <<>> support.example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31557
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;support.example.com.                IN      A

;; ANSWER SECTION:
support.example.com. 300     IN      CNAME   example.zendesk.com.
example.zendesk.com. 300     IN      A       104.16.57.111
example.zendesk.com. 300     IN      A       104.16.55.111

;; Query time: 20 msec
;; SERVER: 10.0.1.1#53(10.0.1.1)
;; WHEN: Wed Oct 06 15:07:24 CEST 2021
;; MSG SIZE  rcvd: 108
```
