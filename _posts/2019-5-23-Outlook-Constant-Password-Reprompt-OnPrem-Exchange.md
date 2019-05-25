---
title: Outlook on Exchange On-Premise Credential Re-prompt
last_modified_at: 2019-05-23T15:16:00-00:00
description: How I fixed a user's password re-prompt issue on Outlook 2016 in an Exchange 2013 environment.
categories:
  - microsoft exchange
tags:
  - microsoft exchange
  - office 2016
  - outlook 2016
  - windows registry
  - powershell
toc: true
toc_sticky: true
toc_label: "On this page"
header:
  # image: /assets/images/image-filename.jpg
  # teaser: /assets/images/logos/PowerShell_5.0_icon_tall.png
excerpt: |
  On a Windows 10 Pro PC, running Outlook 2016, which connects to an Exchange 2013 on-premise server - a user was
  frequently receiving prompts...
---

## The Problem

On a Windows 10 Pro PC running Outlook 2016, connecting to an Exchange 2013 on-premise server - a user was
frequently receiving prompts for their username and password in an Office 365-style Modern Authentication window.

Having confirmed that the user name and password were definitely correct, it was time to start Googling for a
solution, since these sorts of issues rarely occur in isolation...

It took some digging, but I happened across a solution which seemed to fit.

## The Explanation

With thanks to user "tedman" from Spiceworks...

> In the newest version of Office 2016, it looks for Autodiscover records at 365 first, before trying to use SCP
> etc. Older versions try to use SCP and HTTP methods, but now 365 is the default.
>
> This can be disabled with this reg key, which stops Outlook from trying to poll 365 autodiscover records first.
>
> <cite><a href="https://community.spiceworks.com/topic/post/7485098">tedman</a></cite>

## The Fix

The below registry key overrides this new behaviour in Outlook:

```regedit
[HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Outlook\AutoDiscover]
"ExcludeExplicitO365Endpoint"=dword:00000001
```

This key didn't exist, so I added it, then the DWORD with the indicated value.

Then I closed and re-opened Outlook 2016, and the password re-prompt did not reoccur.

Just to be on the safe side, I then rebooted the PC and opened Outlook again - and the issue remained solved.

## The PowerShell Way

If you'd prefer a quicker way to add the above key to the registry, see below:

```powershell
New-Item –Path 'HKCU:\Software\Microsoft\Office\16.0\Outlook\' –Name AutoDiscover

$newPropertyParams = @{
  'Path'  = 'HKCU:\Software\Microsoft\Office\16.0\Outlook\AutoDiscover'
  'Name'  = 'ExcludeExplicitO365Endpoint'
  'Value' = '0x00000001'
  'Type'  = DWORD
}
New-ItemProperty @newPropertyParams
```