---
title: 'ssh'
date: 2021-09-15
tags: [ 'ssh', 'shell', 'network', 'protocol', 'key', 'fingerprint' ]
---

## Usage

### Get SSH key fingerprint

```bash
$ ssh-keygen -lf id_martinsimon.pub
256 SHA256:btO1ftKaZ1AxBLvfuFLS2J4a6Aw6aNStsnB6FB23e7g me@martinsimon.me (ED25519)
```

In newer versions of OpenSSH, Base64 encoded SHA-256 is shown instead of
hexadecimal MD5. To show the legacy style hash, use:

```bash
$ ssh-keygen -l -E md5 -f id_martinsimon.pub
256 MD5:f0:cc:6d:7c:3c:84:15:c1:d0:52:57:c2:c4:1e:c5:09 me@martinsimon.me (ED25519)
```

However, if you’re dealing with the fingerprints that Amazon shows in the EC2
Key Pairs console, unfortunately that may be a different beast. If it’s a
32-digit hex string, it’s the standard MD5 SSH public key fingerprint above.
But if it’s 40 hex digits, it’s actually a fingerprint computed by taking the
SHA1 of the private key in PKCS#8 format:

```bash
$ openssl pkcs8 -in aws_root -nocrypt -topk8 -outform DER | openssl sha1 -c
(stdin)= 82:90:cb:b6:c4:ce:ee:e4:d5:f6:13:76:48:07:49:4d:a9:8d:cc:f2
```

<p style="font-size: 12px" align="right">
    Source: <a href="https://superuser.com/questions/421997/what-is-a-ssh-key-fingerprint-and-how-is-it-generated">superuser</a>
</p>
