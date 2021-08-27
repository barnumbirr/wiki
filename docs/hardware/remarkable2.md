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

The Remarkable's timezone is set to `UTC`.  
Due to unknown reasons, the reMarkable interface does not expose a visible
clock to the user. [remarkable-hacks](#additional-functionality) does expose a
clock to the user, hence why we need to set the correct timezone.

```bash
$ timedatectl set-timezone "Europe/Paris"
```

To see what timezones are available look at the contents of
`/usr/share/zoneinfo`.

## Additional functionality

[remarkable-hacks](https://github.com/ddvk/remarkable-hacks) adds a [literal
metric ton of
additional features](https://github.com/ddvk/remarkable-hacks#quick-doc) to the
Remarkable. Follow the installation instructions in the project's README.
