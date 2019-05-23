---
title: Windows 1903 PowerShell OpenSSH Error 1058
description: I updated my Windows Surface Laptop and my desktop PC to Windows 10 version 1903 today and got an error when starting PowerShell.
categories:
  - powershell
  - openssh
tags:
  - openssh
  - powershell
  - windows 10 1903
  - windows 10
  - windows feature update
toc: true
toc_sticky: true
toc_label: "On this page"
header:
  # image: /assets/images/image-filename.jpg
  # teaser: /assets/images/logos/PowerShell_5.0_icon_tall.png
excerpt: |
  After a smooth feature update from Windows 10 1809 to 1903 on my Surface Laptop and my desktop PC, I started up my development environment...
---

## The Problem

After a smooth feature update from Windows 10 1809 to 1903 on my Surface Laptop and my desktop PC, I started up my development environment on both, because I've had some surprises before after such updates.

On the Surface Laptop, I started Cmder, which spawned the usual PowerShell terminal, then a few seconds later, I was presented by this sequence of messages:

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

unable to start ssh-agent service, error :1058
Error connecting to agent: No such file or directory
Error connecting to agent: No such file or directory
Loading personal and system profiles took 1435ms.
```

My PowerShell Profile, among various other things particular to my environment, checks for and loads my private SSH key, which involves starting the `ssh-agent` service.

As it turned out, somewhere during the update, Windows had "helpfully" set the start type of `ssh-agent` to `Disabled` - as I discovered:

```powershell
> Get-Service ssh-agent | Select-Object StartType


StartType
---------
 Disabled
```

## How I Fixed It

The fix was implicit in the problem - I simply changed the startup type of `ssh-agent` to Automatic:

```powershell
> Set-Service ssh-agent -StartupType Automatic
```

Then started the service:

```powershell
> Start-Service ssh-agent
```

And reloaded Cmder, seeing the rather more encouraging (and familiar) messages below (edited for privacy):

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

Identity added: C:\Users\username/.ssh/id_rsa (C:\Users\username/.ssh/id_rsa)
Loading personal and system profiles took 1353ms.

(Admin) : user@hostname : C:\folder\subfolder : 23/05/2019 17:17:10 :
>
```

## But Wait

This was all well and good, but why was the problem not also occuring on my desktop PC?

First, I confirmed that the ssh-agent startup type somehow escaped the "Disable-Hammer" of the feature update...

```powershell
> Get-Service ssh-agent | Select-Object StartType
Get-Service : Cannot find any service with service name 'ssh-agent'.
At line:1 char:1
+ Get-Service ssh-agent | Select-Object StartType
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ssh-agent:String) [Get-Service], ServiceCommandException
    + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.GetServiceCommand
```

Well, that came as a surprise.

Better check the install status of the OpenSSH Client Optional App...

```powershell
> Get-WindowsCapability -Online -Name *openssh.client*


Name         : OpenSSH.Client~~~~0.0.1.0
State        : NotPresent
DisplayName  : OpenSSH Client
Description  : OpenSSH-based secure shell (SSH) client, for secure key management and access to remote machines.
DownloadSize : 1316207
InstallSize  : 5300763
```

This shook loose some dust from my memory. As I recall, at some point I got annoyed with having to manage the `ssh-agent` service every time a Windows feature update occurred, so I uninstalled it from my desktop PC.

So I removed the OpenSSH Client Optional App from my Surface Laptop:

```powershell
> $capability = Get-WindowsCapability -Online -Name *openssh.client*
> Remove-WindowsCapability -Name $capability.name -Online

Path          :
Online        : True
RestartNeeded : True

>Restart-Computer
```

After the Surface came back online, I opened Cmder and waited for a whole bunch of unexpected errors:

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

Identity added: /c/Users/username/.ssh/id_rsa (/c/Users/username/.ssh/id_rsa)
Loading personal and system profiles took 4305ms.

(Admin) : user@hostname : C:\folder\subfolder : 23/05/2019 17:36:06 :
```

But didn't get any - so I called this a win and I look forward to *not* having to apply the above fix again in future!