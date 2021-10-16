---
title: 'Docker'
date: 2021-08-27
tags: [ 'docker', 'container', 'virtualization' ]
---

## Cleaning up

### Remove all dangling data

Delete all dangling data (i.e. in order: stopped containers, volumes without
containers and images with no containers). Even unused data, with `-a/--all`
option.

```bash
$ docker system prune
```

### Remove dangling containers

```bash
$ docker containers prune
```

### Remove dangling images

```bash
$ docker image prune
```

### Remove dangling volumes

```bash
$ docker volume prune
```

### Remove dangling networks

```bash
$ docker network prune
```

### Remove tagged images older than a month

```bash
$ docker images --no-trunc --format '{{.ID}} {{.CreatedSince}}' \
    | grep ' months' | awk '{ print $1 }' \
    | xargs --no-run-if-empty docker rmi
```

## List images by descending size

```bash
$ docker images --format '{{.Size}}\t{{.Repository}}\t{{.Tag}}\t{{.ID}}' | sed 's/ //' | sort -h -r | column -t
```
