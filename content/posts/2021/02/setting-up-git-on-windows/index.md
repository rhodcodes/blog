---
title: "Setting Up Git & SSH on Windows"
description: "How to set up git to use ssh on Windows 10 and using OpenSSH"
date: 2021-02-16
draft: true
images : ["post-cover.png"]
tags: ["git", "ssh", "windows"]
categories: ["tutorials"]
---

Have you ever had the pleasure of sitting in front of a fresh clean windows installation
and gone to try and remember how you set up [Git](https://git-scm.com/) to work with ssh?
Wasn't there something to do with some silly [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/)?
Or maybe something to do with a [Tortoise](https://tortoisegit.org/)?

I recently had to rebuild my work laptop and had to remember how to set it all up again,
and I remembered that it was a pain in the arse. But with a bit of searching I discovered
that Windows10 now has an inbuilt SSH client which made things a lot easier.

I'm assuming you've got Git installed already at this point.

## Getting going with OpenSSH

Lets start by installing the OpenSSH client into Windows, if it isn't already. Open a
[powershell](https://github.com/PowerShell/PowerShell) [terminal](https://aka.ms/terminal)
with Administrator rights. The command below will install the OpenSSH client.

```ps
Add-WindowsCapability -Online -Name OpenSSH.Client
```

### SSH-Agent

SSH-Agent is a tool that manages your keys for you and runs as a Windows service. By default
Windows disables the service so we'll start it and then set its startup type to Automatic so
that it's always available.

```ps
# (With Admin rights)
Start-Service -Name ssh-agent

Set-Service -Name ssh-agent -StartupType "Automatic"
```

## Creating your SSH Key

If this is your first time using SSH to authenticate with a server then you'll need to create a key.

A key comes in to two parts - public and private. You give the public part to the server, and keep the
private part to yourself. If you're after a more in depth explanation about how it all works then Digital
Ocean have a good article about it
[here](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process).

We'll create two types of keys:

1. **Ed25519** - Uses elliptic curve cryptography (I have no idea what it means, but it sounds good) to produce
a short key that's quick to process but tough to break and it's the recommended key type to use when you can.
2. **RSA** - The previous default and if you've created keys in the past then it's probably been this type.
You can pick your key size, but anything below 2048 bits is **not** recommended.

I'm including the RSA key here because not everywhere supports using the Ed25519 key so we'll generate both
so you can fall back to the RSA key if that's the case. [Azure Devops](https://devops.azure.com) - I'm looking
at you.


```ps
# Ed25519
ssh-keygen -t ed25519 -C "user@host"

# RSA with a 4096 bit key
ssh-keygen -t rsa -b 4096 -C "user@host"
```

The text after the `-C` is a comment and is usually used to append an email or some form of identifier.
I tend to use my username and the machine name for this as I generate one key per machine.

When it prompts you to enter a file in which to save the key, it's best to hit return and let it save it
using its default values which is in `.ssh/` in the root of your user directory. 

When running the command above it will ask you for a passphrase and where to store the private and public
key files.

If you want to secure your key further with a passphrase you can, but you'll be asked for it every time
it's used. I just hit enter and leave it blank which makes it easier to use in scripts and automated
environments if needed.

### Add keys to key chain

Next we add the keys to ssh-agent so that the service can present them when asked by SSH.
If you've used the default location and file names in the previous step then you just run `ssh-add`
which will pick up the key files in `.shh/` and print a success message to screen.

```ps
ssh-add
```

## Fixing the Environment

For Git to know which ssh binary to use, which in turn uses the correct ssh-agent to provide our key files
we need to set an environmental variable - `$GIT_SSH` so that it points to ssh.exe which lives in
`C:\Windows\System32\OpenSSH\ssh.exe`.

You might have this already set if you've used PuTTY or TortoiseSVN as this is how they tell git to use
their own SSH binaries.

Using the powershell below will set the environmental variable to the correct value or create it if
it doesn't already exist.

```ps
[System.Environment]::SetEnvironmentVariable('GIT_SSH','C:\Windows\System32\OpenSSH\ssh.exe')
```

## Pre-launch

Sometimes when using git from an external GUI such as [Fork](https://git-fork.com/) it will appear to
hang or error out when doing any commands involving SSH. In my experience this is when you connect to
a server that you've not connected to before and SSH doesn't yet trust it's identity.

Before using any git commands I like to run a manual ssh connection to any Git servers that I'll be
interacting. To get the details we need for this get the SSH details provided by a repo and take the
bits before the `:`. This is the user and the host we need for SSH.

For example `git clone git@github.com:rhodcodes/blog.git` is how I'd clone the repo for this blog.
And `git@github.com` is the user and the host we need to manually connect via SSH.

```ps
ssh git@github.com

The authenticity of host `github.com (140.82.121.4)' cant be established.
RSA Key Fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

If we answer `yes` then SSH will add the host to the `known_hosts` file in `.ssh/` and it will ensure
that we trust any future connections to that host with that particular key fingerprint.

Here are the user and hosts for manually connecting to some of the major Git Hosts

```
GitHub:         git@github.com
GitLab:         git@gitlab.com
Azure DevOps:   git@ssh.dev.azure.com
```

And there we go. Happy gitting and happy shhing.
