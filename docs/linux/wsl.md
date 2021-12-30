---
title: 'WSL'
date: 2021-12-30
tags: [ 'Linux', 'Debian', 'Windows' ]
---

# Windows Subsystem for Linux (WSL)

## Installation

### Git

`WSLGit` provides a small executable that forwards all arguments to `git`
running inside Bash on Windows/Windows Subsystem for Linux (WSL).

```bash
$ wget -P /mnt/c/Program\ Files/Git/cmd -O git.exe https://github.com/andy-5/wslgit/releases/download/v1.0.1/wslgit.exe
```

## Configuration

### Terminal profile icons

Place the icon image `debian.png` in the folder located at
`%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\RoamingState`,
then add the following line to your Windows Terminal `settings.json`:

```
"icon": "ms-appdata:///roaming/debian.png",
```

### SSH key management

In (Admin) Powershell:

```
Get-Service -Name ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Then from any user shell:

```
ssh-add /path/to/.ssh/id_rsa
```
