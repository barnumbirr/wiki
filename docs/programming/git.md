---
title: "Git"
date: 2021-08-25
tags: [ 'Git', 'Github', 'repository', 'branch' ]
---

## Configuration

### Setting up multiple Git profiles

When using git as the version control tool for both personal and work related
projects, it might be interesting to isolate the two (or more) profiles.

In your home directory, create the following files:

```bash
$ ls -la
-rw-rw-rw-  1 root root  483 Aug 25 08:07 .gitconfig
-rw-rw-rw-  1 root root   57 Apr  2  2021 .gitconfig-personal
-rw-r--r--  1 root root   57 Aug  5  2021 .gitconfig-work
```

Add the following to `.gitconfig`:

```bash
[includeIf "gitdir:/path/to/personal/projects/directory"]
    path = .gitconfig-personal
[includeIf "gitdir:/path/to/work/projects/directory"]
    path = .gitconfig-work
[...]
```

Each subsequent configuration file can then define it's own user and Git
settings, e.g.:

```bash
$ cat .gitconfig-personal
[user]
  name = John Doe
  email = john.doe@personal.com
[help]
  autocorrect = 1
$ cat .gitconfig-work
[user]
  name = John Doe
  email = john.doe@work.com
[diff]
  tool = icdiff
[difftool]
  prompt = false
[icdiff]
  options = --highlight --line-numbers
```

### Automatically sign commits

On Debian:

```bash
$ apt-get install gnupg2
$ gpg2 --import file.asc
$ gpg2 --list-secret-keys --keyid-format=long
/root/.gnupg/pubring.kbx
------------------------
sec   rsa4096/632C9BB6CF21205A 2015-09-30 [SC]
      6F2B0C373EBE0DD7975D51B9632C2AA6CF21205A
uid                 [ unknown] John Doe <john.doe@personal.com>
ssb   rsa4096/9C04F25B59B68D59 2015-09-30 [E]

$ eval "$(keychain --stop others --quiet --quick --eval --agents gpg,ssh\
    --inherit any --timeout 31622400 ~/.ssh/id_example1 ~/.ssh/id_example2\
    ~/.ssh/id_example3 632C9BB6CF21205A)"
$ echo "test" | gpg2 --clearsign
```

Add the following to `.gitconfig`:

```bash
[commit]
  gpgsign = true
[gpg]
  program = gpg2
```

and the following to `.gitconfig-xxxxxxx`

```bash
  signingkey = 632C9BB6CF21205A
```

## Usage

### Squash last `<n>` commits

If you want to write the new commit message from scratch, this suffices:

```bash
$ git reset --soft HEAD~<n>
$ git commit
$ git push -f
```

If you want to start editing the new commit message with a concatenation of the existing commit messages (i.e. similar to what a\
pick/squash/squash/…/squash `git rebase -i` instruction list would start you with), then you need to extract those messages and pass them to `git commit`:

```bash
$ git reset --soft HEAD~<n>
$ git commit --edit -m"$(git log --format=%B --reverse HEAD..HEAD@{1})"
$ git push -f
```

<p style="font-size: 10px" align="right">
    Source: <a href="https://stackoverflow.com/a/5201642">Stackoverflow</a>
</p>

### Squash all commits into one

Make sure everything is committed, and write down the latest ```commit_id``` in case something goes wrong, or create a separate branch as a backup.

Reset your head to the first commit, but leave your index unchanged. All changes since the first commit will now appear ready to be committed.

```bash
$ git reset --soft `git rev-list --max-parents=0 --abbrev-commit HEAD`
```

Amend your commit to the first commit and change the commit message.

```bash
$ git commit --amend -m "initial commit"

# if you want to keep the existing commit message
$ git commit --amend --no-edit
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
$ git tag -d <tag_name>
```

For a remote tag (if you've already pushed the tag)

```bash
$ git push --delete origin <tag_name>
```

### Syncing a fork

```bash
$ git fetch upstream
$ git checkout master
$ git merge upstream/master
```
