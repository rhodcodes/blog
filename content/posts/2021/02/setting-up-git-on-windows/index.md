---
title: "Setting Up Git & SSH on Windows"
date: 2021-02-16T20:32:39Z
draft: true
images : ["post-cover.png"]
tags: ["git", "ssh", "windows"]
categories: ["tutorials"]
---

This is a post all about my git got flipped and I sorted it out.

## Install OpenSSH on Windows

## ssh-agent auto start

```ps
Needs admin
# Starts ssh-agent on boot
Set-Service -Name ssh-agent -StartupType "Automatic"
# To start it this time
Start-Service -Name ssh-agent

```

## Create SSH Key

existing keys should live in `~/.ssh`

### Ed25519

```ps
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### RSA (For older legacy systems)

```ps
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Hit enter to all prompts for accept defaults.

### Add keys to key chain

```ps
ssh-add # scans /.ssh for keys and adds to key-agent
```

## Set Enviromental Variables
Might need to set `$GIT_SSH`

```ps
[System.Environment]::SetEnvironmentVariable('GIT_SSH','C:\Windows\System32\OpenSSH\ssh.exe')
```

When using git to pull from a host for the first time when using a GUI it
might hang. Its because the hostname isn't in the `known_hosts` file. Fix
this by sshing into the host manually. Using github as an example

```ps
ssh git@github.com # git@github.com <-- this bit :rhodcodes/blog.git

The authenticity of host `github.com (140.82.121.4)' cant be established.
RSA Key Fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
Type yes and be done with it.