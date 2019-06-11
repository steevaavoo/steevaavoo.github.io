---
title: Office 2016 Apps Fail with Error 0xC0000142
last_modified_at: 2019-06-11T10:36:00-00:00
description: How to fix the 0xC0000142 error when opening Microsoft Office 2016 "click-to-run" apps
categories:
  - office 365
tags:
  - microsoft update
  - error 0xC0000142
  - windows
  - office 2016
toc: true
toc_sticky: true
toc_label: "On this page"
header:
  # image: /assets/images/image-filename.jpg
  # teaser: /assets/images/logos/PowerShell_5.0_icon_tall.png
excerpt: |
  Microsoft Office 2016 apps based on Office 365 "click-to-run" fail to open after a long delay, then finally show
  error code 0xC0000142...
---

## The Challenge

Microsoft Office 2016 apps based on Office 365 "click-to-run" fail to open after a long delay, then finally show
error code 0xC0000142.

Microsoft's Support pages have a record of the issue, and a "solution":

> **ISSUE**
>
> When you try to start an Office 2016 app, such as Excel 2016 or Word 2016, it fails and you get error code
> 0xC0000142.
>
> **STATUS: FIXED**
>
> This issue is fixed in Monthly Channel Version 1803 (Build 9126.2116) or greater. To get the latest update
> immediately, open an Office app and choose **File > Account > Update Options > Update Now**.
>
> <cite><a href="https://support.office.com/en-us/article/error-0xc0000142-when-you-start-an-office-2016-application-0249adb1-06d7-41a3-b2df-da5ff57fab37">Microsoft</a></cite>

Okay, so let me get this crystal clear...

Open an Office App and Update Now.

When the issue being fixed is an *inability* to open *any* office apps...

Maybe I'm missing something?

## How I Fixed It

Since the proposed solution from Microsoft is impossible, I did an Online Repair of Office.

- Windows Key + R
- Type `appwiz.cpl` then hit Enter
- Find Microsoft Office 365 and click to highlight it
- Click "Change"
- Choose "Online Repair"
- Click "Repair"

After a while, Office should be completely re-installed with the latest binaries, and all should work as expected.
