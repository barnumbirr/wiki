---
title: 'Remarkable 2'
date: 2021-08-27
tags: [ 'remarkable', 'paper', 'tablet', 'e-ink' ]
---

The Remarkable 2 or why you won't ever need paper notebooks again.

## Change SSH password

The default IP and SSH password of the Remarkable tablet is shown in the
'GPLv3 Compliance' section of the 'Copyrights and licenses' page.  
Navigate to it as follows: `Menu > Settings > Help > Copyrights and licenses > 
General information > GPLv3 Compliance`.

The password is stored in `/etc/remarkable.conf`.

```bash
[General]
DeveloperPassword=password
```

## Updating timezone

Due to unknown reasons, the reMarkable interface does not expose a visible
clock to the user.

```bash
$ timedatectl set-timezone "Europe/Paris"
```

To see what timezones are available look at the contents of
`/usr/share/zoneinfo`.
