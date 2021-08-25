---
title: "Git"
date: 2021-08-25
tags: [ "Git", 'TLS', 'certificate' ]
---

## Usage

### Squash last `<n>` commits

If you want to write the new commit message from scratch, this suffices:

```bash
user@host:~$ git reset --soft HEAD~<n>
user@host:~$ git commit
user@host:~$ git push -f
```

If you want to start editing the new commit message with a concatenation of the existing commit messages (i.e. similar to what a\
pick/squash/squash/…/squash `git rebase -i` instruction list would start you with), then you need to extract those messages and pass them to `git commit`:

```bash
user@host:~$ git reset --soft HEAD~<n>
user@host:~$ git commit --edit -m"$(git log --format=%B --reverse HEAD..HEAD@{1})"
user@host:~$ git push -f
```

<p style="font-size: 10px" align="right">
    Source: <a href="https://stackoverflow.com/a/5201642">Stackoverflow</a>
</p>

### Squash all commits into one

Make sure everything is committed, and write down the latest ```commit_id``` in case something goes wrong, or create a separate branch as a backup.

Reset your head to the first commit, but leave your index unchanged. All changes since the first commit will now appear ready to be committed.

```bash
user@host:~$ git reset --soft `git rev-list --max-parents=0 --abbrev-commit HEAD`
```

Amend your commit to the first commit and change the commit message.

```bash
user@host:~$ git commit --amend -m "initial commit"

# if you want to keep the existing commit message
user@host:~$ git commit --amend --no-edit
```

Force push your changes.

```bash
user@host:~$ git push -f
```

<p style="font-size: 10px" align="right">
    Source: <a href="https://stackoverflow.com/a/49900667">Stackoverflow</a>
</p>

### Delete tags

For a local tag, use

```bash
user@host:~$ git tag -d <tag_name>
```

For a remote tag (if you've already pushed the tag)

```bash
user@host:~$ git push --delete origin <tag_name>
```

### Syncing a fork

```bash
user@host:~$ git fetch upstream
user@host:~$ git checkout master
user@host:~$ git merge upstream/master
```
