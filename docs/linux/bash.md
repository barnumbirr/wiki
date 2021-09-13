---
title: 'bash'
date: 2021-08-30
tags: [ 'bash', 'shell' ]
---

## Temporarily disable alias

Let's assume you've set up a neat `ls` alias:

```bash
alias ls='ls -lah --color=auto'
```

To temporarily disable said alias, prepend your command with `\`:

```bash
$ ls
total 12K
drwxrwxrwx 1 root root  512 Aug 28 18:28 .
drwxrwxrwx 1 root root  512 Aug 28 16:15 ..
drwxrwxrwx 1 root root  512 Aug 28 02:19 docs
drwxrwxrwx 1 root root  512 Aug 30  2021 .git
drwxrwxrwx 1 root root  512 Aug 25 17:51 .github
-rwxrwxrwx 1 root root 6.9K Aug 25 17:51 LICENSE
-rwxrwxrwx 1 root root 1.3K Aug 29 16:16 mkdocs.yml
drwxrwxrwx 1 root root  512 Aug 29 16:19 overrides
-rwxrwxrwx 1 root root  399 Aug 30 03:03 README.md
-rwxrwxrwx 1 root root   42 Aug 27 23:58 requirements.txt
$ \ls
docs  LICENSE  mkdocs.yml  overrides  README.md  requirements.txt
```
