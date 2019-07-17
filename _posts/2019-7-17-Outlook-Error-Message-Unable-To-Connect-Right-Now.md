---
title: 
last_modified_at: 2019-07-17T11:00:00-00:00
description: 
categories:
  - Outlook
  - Office
tags:
  - Outlook
  - Office 365
  - Networking
  - Windows
toc: true
toc_sticky: true
toc_label: "On this page"
header:
  # image: /assets/images/image-filename.jpg
  # teaser: /assets/images/logos/PowerShell_5.0_icon_tall.png
excerpt: |
  We are unable to connect right now. Please check your network and try again later.
---

## The Problem

A client of mine was receiving the below error every time they tried to either activate Outlook or connect Outlook
to their Exchange Online account:

`We are unable to connect right now. Please check your network and try again later.`

## How I Fixed it

After trying all the "usual suspects" - Office Update, Windows Update, Online Repair of Office, disabling their
software VPN etc. - none of which made any difference, I did a fair amount of Googling, and happened across a
helpful solution, as below:

1. Open **Network & Sharing Centre**
   1. You should see that the network description shows *"Unknown"* and reports *"The service to detect this*
*status is disabled"*.
1. Set the startup type of the Network List service to Automatic and start it.
   1. This might fail, stating that a dependency is not running - if that's the case, set to Automatic startup type
(if necessary) and start the Network Location Awareness service, and try the above again.

It would appear that in this case there was a pretty decent clue in the error message itself...

### The PowerShell way...

```PowerShell
Set-Service -ServiceName NlaSvc -StartupType Automatic
Set-Service -ServiceName netprofm -StartupType Automatic
Start-Service -ServiceName NlaSvc,netprofm
```
