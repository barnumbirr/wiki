---
title: 'rutorrent'
date: 2021-09-12
tags: [ 'rutorrent', 'web' ]
---

## Configuration

### Moving torrents upon completion

`rutorrent` can automate moving torrents upon completion using the
[Autotools](https://github.com/Novik/ruTorrent/wiki/PluginAutotools) plugin.
Moving torrents may come in handy in when using multiple libraries in
`Plex Media Server`.

1. In `rutorrent`, choose `Settigs > Autotools`.
2. Enable "Autolabel" feature, Template: `{DIR}`
3. Enable "AutoMove" if torrent's label matches filter:
`/ebooks|movies|music|series/`
4. Set path to finished downloads: `/share/data/torrents`, operation type `Move`
and enable `Add torrent's label to path`
