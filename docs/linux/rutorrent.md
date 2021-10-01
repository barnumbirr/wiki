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

1. In `rutorrent`, choose `Settings > Autotools`.
2. Enable "Autolabel" feature, Template: `{DIR}`
3. Enable "AutoMove" if torrent's label matches filter:
`/apps|ebooks|movies|music|series/`
4. Set path to finished downloads: `/share/data/torrents`, operation type `Move`
and enable `Add torrent's label to path`

### Fix missing label icons

The [tracklabels plugin](https://github.com/Novik/ruTorrent/tree/master/plugins/tracklabels)
stores it's 16x16 pixel .png images under `/rutorrent/plugins/tracklabels/labels/`.  
Adding missing label icons is as easy as copying an existing icon to a different
name:

```bash
$ cp /opt/share/www/rutorrent/plugins/tracklabels/labels/movie.png /opt/share/www/rutorrent/plugins/tracklabels/labels/movies.png
$ cp /opt/share/www/rutorrent/plugins/tracklabels/labels/books.png /opt/share/www/rutorrent/plugins/tracklabels/labels/ebooks.png
```
