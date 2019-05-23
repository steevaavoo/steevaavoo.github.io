---
title: Get Exchange Mailbox Primary Email Addresses with PowerShell
last_modified_at: 2019-05-23T09:51:00-00:00
description: How I extracted and isolated the primary SMTP address for mailboxes in an Exchange Online tenant.
categories:
  - powershell
tags:
  - microsoft exchange
  - exchange online
  - powershell
  - office 365
toc: true
toc_sticky: true
toc_label: "On this page"
header:
  # image: /assets/images/image-filename.jpg
  # teaser: /assets/images/logos/PowerShell_5.0_icon_tall.png
excerpt: |
  I needed to create a list of all Primary SMTP addresses for all users on a client's Exchange Online tenant...
---

## The Challenge

I needed to create a list of all Primary SMTP addresses for all users on a client's Exchange Online tenant, to compare with addresses registered in their email filtering system and remove invalid accounts.

`Get-Mailbox` and `Get-Recipient` both return an EmailAddresses property for each object, but they are a serialised combination of all types of address registered against every mailbox returned.

I wanted to simply return the Primary SMTP address (which is denoted by the capitalised “SMTP:” before the address.)

## How I Did It

First I wanted to see how I could manipulate the EmailAddresses Property contents and filter out exactly what I needed, so first, in order to save time querying the tenant over and over I grabbed the `Get-Mailbox` results into a variable.

`$mailboxes = Get-Mailbox`

I quickly realised that I only wanted to work on a single object, so I re-ran the above but selected just the first 1.

`$mailboxes = Get-Mailbox | Select-Object -First 1`

I took a peek at the `EmailAddresses` property and got what I was hoping for (edited for privacy):

```powershell
PS:\>$mailboxes.EmailAddresses

smtp:email.address@tenant.onmicrosoft.com
SMTP:email.address@ourdomain.com
X500:/o=First Organization/ou=OUName (RANDOMSTRING)/cn=Recipients/cn=RANDOMSTRING
x500:/o=First Organization/ou=OUName (RANDOMSTRING)/cn=Recipients/cn=RANDOMSTRING
```

I then ran the object through a couple of `Where-Object` cmdlets to see how I could isolate the SMTP: address:

```powershell
PS:\>$emailaddresses = $mailboxes.emailaddress | Where-Object { $_ -like ‘*SMTP*’ }

smtp:email.address@tenant.onmicrosoft.com
SMTP:email.address@ourdomain.com
```

Close, but too many results… Because I didn’t consider casing! Solved by simply adding a "c" to the beginning of the comparison operator (`-like` becomes `-clike`)...

```powershell
PS:\>$emailaddresses = $mailboxes.emailaddress | Where-Object { $_ -clike ‘*SMTP*’ }

SMTP:email.address@ourdomain.com
```

Suitably pleased with the result, I then needed to expand this operation to run against all the objects returned from `Get-Mailbox` - which meant getting my hands dirty in VSCode... Yay!

## Building the Function

This started out innocently enough as a simple script, but as you'll see, it began to evolve into a handy function as I saw potential applications for it with other clients.

I needed to return an object containing just the user name and the primary SMTP address, so I employed a couple of `foreach` loops:

```powershell
        foreach ($mailbox in $mailboxes) {
            $emailaddresses = $mailbox.EmailAddresses
            foreach ($emailaddress in $emailaddresses) {
                $smtpaddress = $emailaddress | Where-Object { $_ -clike '*SMTP*' }
                [PSCustomObject]@{
                    'Name'        = $mailbox.Name
                    'SMTPAddress' = $smtpaddress
                }
            } #foreach emailaddress
        } #foreach recipient
```

This returned a list of 400+ objects exactly as desired (edited for privacy and brevity):

```powershell
Name                                                         SMTPAddress
----                                                         -----------
User.1                                                       SMTP:user.1@domain.com
User.2                                                       SMTP:user.2@domain.com
User.3                                                       SMTP:user.3@domain.com
User.4                                                       SMTP:user.4@domain.com
User.5                                                       SMTP:user.5@domain.com
etc
```

At this point, it seemed logical to upscale this into a function - so I created the appropriate structure, added a `try`/`catch` to deal with the possibility that the user has not connected to Exchange Online / Exchange on-premise, then added an `if`/`else` construct to filter out the inevitable blank results due to the nested `foreach`/`where-object` process.

The results are in my GitHub [repo](https://github.com/steevaavoo/GetsbExoPrimaryEmail). The module is now a working work-in-progress and my intentions are noted in the Projects section.

Please feel free to comment and make suggestions - there's almost certainly something I can do to refine this and I'm always keen to get tips. I'd also love to hear if this has helped anyone in their early steps into scripting and automation with PowerShell :)