---
title: Enable Azure Bash Cloud Shell in VSCode
description: How to enable the Microsoft Azure Cloud Bash Shell in Visual Studio Code
categories:
  - azure cloud shell
tags:
  - azure
  - azure cloud shell
  - bash
  - vscode
toc: true
toc_sticky: true
toc_label: "On this page"
header:
  # image: /assets/images/image-filename.jpg
  # teaser: /assets/images/logos/PowerShell_5.0_icon_tall.png
excerpt: |
  Usually a couple of lines from the top of the post...
---

## The Challenge

Whilst running through an Azure DevOps Kubernetes [Lab](https://azuredevopslabs.com/labs/vstsextend/kubernetes/),
I decided I'd rather use the terminal in VSCode to run the necessary code snippets, so I could more easily copy,
 paste and individualise the requisite variables as I went through.

## How I Did It

Step 1 was to install the "Azure Account" extension in VSCode:

```powershell
code --install-extension ms-vscode.azure-account
```

Then when I hit F1 in VSCode to get the Command Palette up and typed "bash", the command `Azure: Open bash in cloud
shell` was available, however, when I clicked it, I got a message saying: "Opening a Cloud Shell currently requires
Node.js 6 or later to be installed."

Therefore, step 2 was to install node.js (I did it with chocolatey, it can also be found through the link offered
by VSCode when you try to run the Azure command without node.js):

```powershell
choco install nodejs -y
```

Since I had VSCode open, I had to close and re-open it.

After the installation succeeded, I tried F1 and "bash" then `Azure: Open bash in cloud
shell` again...

After providing my Azure login credentials in the authentication window which spawned, I got the result I
was hoping for.

```bash
Requesting a Cloud Shell...
Connecting terminal...
username@Azure:~$
```