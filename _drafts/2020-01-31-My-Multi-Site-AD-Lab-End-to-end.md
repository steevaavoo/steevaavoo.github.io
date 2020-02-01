---
title: 
last_modified_at: 2020-01-31T17:00:00-00:00
description: 
categories:
  - hyper-v
tags:
  - virtualisation
  - lab
  - active directory
  - ad sites & services
  - vyos
  - windows server
toc: true
toc_sticky: true
toc_label: "On this page"
header:
  # image: ![ALT-TEXT](/assets/images/image-filename.jpg)
  # teaser: /assets/images/logos/PowerShell_5.0_icon_tall.png
excerpt: |
  I wanted to simulate a multi-site Active Directory installation I had coming up. From the inter-site WAN connections
  through to the AD Sites and Services topology configuration...
---

## The Challenge

I wanted to simulate a multi-site Active Directory installation I had coming up. From the inter-site WAN connections
through to the AD Sites and Services topology configuration.

As a fan of VMWare VSphere and with little experience of Hyper-V beyond conversations, I figured this was an opportunity
to familiarise myself with that technology too.

## How I Did It

### Gathering Resources

#### Windows Server DVD Images

I headed over to the [Microsoft Evaluation Centre](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019)
to get ISOs for Server 2019 and Server 2012 R2.


#### VyOS

To simulate the inter-site routing I decided to use an old favourite - the rolling release version of
[VYoS](https://downloads.vyos.io/rolling/current/amd64/vyos-1.3-rolling-202001160217-amd64.iso).
The version I used for this lab was 1.3-rolling-202001160217.

### Setting up the Lab

#### Enabling Hyper-V

Thanks to Docker Desktop, I already had the Hyper-V role and management tools installed, but if you want to switch
it on real quick in Windows 10 Pro (assuming your system meets the relevant pre-requisites), you can use the following
PowerShell script:

```powershell
Enable-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V -Online -Verbose -All
Restart-Computer
```

#### Customising Hyper-V

Just for ease of use, I modified 2 settings in Hyper-V Manager for my lab:

Default VM and VHD locations - both set to (in my case) D:\VMs

Start Hyper-V Manager if it's not already open.

![Hyper-V Icon](/assets/images/Hyper-V-Manager-Icon.png)

Right-click the name of your computer in Hyper-V manager, and click Hyper-V Settings...

Change the default Virtual Machines and Virtual Hard Disks folders and click OK.

![Hyper-V-Settings](/assets/images/Hyper-V-Settings.png)


#### Windows Server Base VMs

Next I set up 2 VMs, one for Windows 2019 Datacenter and one for Server 2012 R2.

I won't go into detail about the install process for Windows Server - that's outside the scope of this guide - I'll
just touch on any non-default settings conducive to my intended lab scenario.

##### Quick summary of settings:

- Location: D:\VMs\\[VM-Name]
- Virtual Machine Generation 2
- Network Connection: Default Switch (to allow downloading programs and updates)
- Virtual Hard Disk Size: 40GB
- Install an operating system from a bootable image file

##### Sidebar: Disabling Automatic Checkpoints

Normally, at least at time of writing, in Win 10 Pro 1909, all newly-created VMs have "Automatic Checkpoints"
enabled by default, and there is no way to globally disable this, so you need to make sure to remember to go to
the settings of each VM you create and disable the feature. You don't _have_ to disable it, but I prefer to choose
when I make checkpoints, so I always disable the automatic ones before starting a VM.

![Automatic Checkpoints Example](/assets/images/Automatic-Checkpoints.png)

#### Windows Server post-install steps

I ran all updates, added a few programs I like to have installed (such as VS Code, for testing and running scripts),
then sysprepped both boxes, shutting them down in OOBE mode.

Next I found the VHDX files for the 2 base VMs, and set them to read-only, to make sure they couldn't be changed
after I started creating Differencing Disks with them as the source disks.

### Setting up the Sites

Before setting up the actual VMs, I wanted to get the virtual sites and routing configured - so here's how:

I made a plan! First I drew out my overall intention and the subnets I would use for each of the virtual sites,
naming each for consistency and ease of identification throughout the process. The first drawing was very abstract,
detail was added as the process continued.

![Topology Abstract](/assets/images/Topology-Abstract.png)

#### Hyper-V Virtual Switches

In Hyper-V Manager, click Virtual Switches Manager... over on the shortcuts list on the right-hand-side.

![Manage Virtual Switches Link](/assets/images/Virtual-Switch-Manager-Shortcut.png)

There's already a "Default Switch" present - later, during VyOS configuration, that will be used as the Internet route.

Click "New virtual network switch" at the top, click to highlight "Internal", click "Create Virtual Switch", name it
after the site, leaving all other settings as default, then click "New virtual network switch" again, repeat
the process until you have all the sites set up, then click OK.

![New Virtual Switch Link](/assets/images/New-Virtual-Switch.png)

![Name Virtual Switch](/assets/images/Name-Virtual-Switch.png)

Ultimately, you should end up with a list of appropriately-named switches like this:

![Virtual Switches](/assets/images/Virtual-Switches.png)

#### VyOS

Next I set up VyOS to handle the inter-site routing.

##### VM Setup

Back in Hyper-V Manager, create a new VM, observing the below non-default options.

##### Quick summary of settings:

- Location: D:\VMs\\[VM-Name]
- Virtual Machine Generation 1
- Startup Memory: 512MB
- Uncheck "Use Dynamic Memory for this virtual machine" (if you don't, VyOS seems to consume all available system
memory over time)
- Network Connection: Default Switch (to allow downloading programs and updates)
- Virtual Hard Disk Size: 2GB
- Install an operating system from a bootable image file

##### Adding Network Adapters

After the New Virtual Machine wizard is finished, right-click the VyOS VM and click Settings...

Click Add Hardware, then choose Network Adapter and click Add:

![Add Hardware to VyOS](/assets/images/VyOS-Add-Hardware.png)

From the Virtual Switch drop-down, choose the Virtual Switch you want the Network Adapter connected to.

Repeat this process until you have added a Network Adapter and connected it to each of the named sites. In this
case, London, Edinburgh and Madrid.

Click Apply. Your VyOS hardware summary should look similar to this:

![VyOS Hardware Summary](/assets/images/VyOs-Hardware-Summary.png)

Leave this Settings window for VyOS open - you'll need some information from it when you set up the ethernet
adapters inside VyOS.

##### Booting the VyOS installer

Start the VM and open its console.

Click in the window and press enter to choose "Live (amd64-vyos)" - if you don't, it will start by default anyway.

Go back to the Settings window for VyOS and click refresh at the top.

![Refresh](/assets/images/Refresh.png)

Next, expand the Network Adapters for all 4 switches (include the Default one), then highlight Advanced Features,
updating the network topology drawing with the last 2 digits of the MAC address shown - this will make it easier
to identify the ethernet adapters and name them in VyOS shortly.

![Topology with MAC Addresses](/assets/images/Topology-Macs.png)

Now close the VyOS Settings window.

##### Logging in and installing VyOS

Back in the VyOS console, log in with `vyos` as both the user name and the password.

Type `install image` and press `ENTER`

`Yes` you would like to continue `ENTER`

Partition `Auto` `ENTER`

Install the image on `sda` `ENTER`

Continue `yes` `ENTER`

Partition size - just press `ENTER` to accept the default.

Image name - again, accept the default by pressing `ENTER`

Accept the default configuration file with `ENTER`

Create a good password for the `vyos` administrator account and press `ENTER` - *be aware that the keyboard layout
in the VyOS console is EN-US*.

Accept the default for the GRUB boot partition with `ENTER`

Type `poweroff` and hit `ENTER`, then type `y` and hit `ENTER` to shut down VyOS.

Bring up the Settings window for VyOS again, then click the DVD Drive entry and change the Media to "None" and
click OK.

Start VyOS again and open the console.

This time you should see a longer list of boot options - choose the first one ending in "(KVM console)"

##### Configuring VyOS

Login with `vyos` and the password you specified earlier.

At the `vyos@vyos:~$` prompt, type `configure` and press `ENTER` - the prompt will change to `vyos@vyos#`

##### Network Interfaces

Type `show interfaces` and press `ENTER` - you will get a list of the ethernet interfaces and their MAC addresses.

![VyOS Interfaces](/assets/images/vyos-interfaces.png)

Take this opportunity to update your network topology diagram.

![Final Topology](/assets/images/Final-Topology.png)

Keeping that handy, you can now start configuring the ethernet interfaces.

Use the below sequence of commands to set up the interfaces with meaningful descriptions and IP addresses. After
this initial setup, you'll enable the SSH server and connect to VyOS using PuTTY (or similar) which will allow you
to copy/paste commands, saving some time.

For some ease, you can use the up-arrow cursor key to bring up previous commands and just modify the bits you need.

Also for ease, VyOS supports tab auto-completion, so you can hit `TAB` for hints if you can't remember a sub-command,
or type in a few letters and press `TAB` to save typing.

```bash
set interfaces ethernet eth0 description 'Internet'
set interfaces ethernet eth0 address dhcp
commit
save
```

NOTE: `commit` will commit any changes you've made to the current session, but if you reboot the router, they will
not be saved. `save` actually writes those changes to the config. This is useful to know if you want to test something out with an option to revert if you made a mistake.

If you type `ping 8.8.8.8` and press `ENTER` you should get replies, meaning you've connected VyOS to the internet
successfully.

Pressing `CTRL+C` will stop the pings.

Now set up the remaining interfaces as below - this will give VyOS an IP ending in .250 on each network.

```bash
set interfaces ethernet eth1 description 'London'
set interfaces ethernet eth1 address 192.168.2.250/24
set interfaces ethernet eth2 description 'Edinburgh'
set interfaces ethernet eth2 address 192.168.3.250/24
set interfaces ethernet eth3 description 'Madrid'
set interfaces ethernet eth3 address 192.168.4.250/24
commit
save
```

If you type `show interfaces` and hit `ENTER` again, you should see something like this:

![VyOS Interfaces Customised](/assets/images/vyos-interfaces-customised.png)

##### Connect via SSH

Now you can prepare VyOS to accept incoming connections on SSH.

Type the following commands:

```bash
set service ssh port 22
commit
save
```

Bring up your Network Adapters on your local Windows machine, and pick one of the adapters named after one of the
sites you've created.

Assign it an IP address on the appropriate subnet for that site (like 192.168.2.249 for London, for example), don't
bother with Default Gateway or DNS servers.

Open up your preferred TTY program (PuTTY in my case) and connect to the .250 address of VyOS on the subnet you
configured an address for in the previous step.

You may need to click Yes to a security thumbprint dialog. Obviously we would never be so ignorant of security in
production.

Enjoy the ability to resize the console window to your heart's content(!)

Log in to VyOS with your username and password - once again note that when you configured the password for VyOS
initially, it was in an EN-US keyboard layout.

Type `configure` at the `vyos@vyos:~$` prompt and press `ENTER`

Now you can copy and paste commands in to VyOS.

##### General VyOS Settings

