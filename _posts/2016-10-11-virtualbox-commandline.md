---
layout:            post
title:             "Virtualbox's little secret: command-line"
menutitle:         "Virtualbox's little secret: command-line"
date:              2016-10-11 00:40:00 +0300
tags:              virtualization virtualbox
category:          Virtualization
author:            bharath
cover:             /assets/header_bg.jpg
published:         true
cover:             /assets/header_bg.jpg
language:          EN
comments:          true
---

We often run into features on some software that are little known but are very handy. Virtualbox has one such feature, the command-line.

**VBoxManage** is the command-line interface to VirtualBox. With it, you can completely control VirtualBox from the command line of
your host operating system. VBoxManage supports all the features that the GUI gives you access to, but it supports a lot more than that. It exposes really all the features of the virtualization engine, even those that cannot (yet) be accessed from the GUI.

Fortunately, VBoxManage has extensive documentation which makes life easy. It covers every available option that there is in VBoxManage. If you ever find yourself using VBoxManage, the docs are your go-to reference.

[VBoxManage documentation](https://www.virtualbox.org/manual/ch08.html)


Rather than going over what\'s already covered in documentation extensively and making this article an yet-another tutorial on VBoxManage, I\'ll go over a problem that I solved using virtualbox command-line recently.

###### The problem at hand

I started coducting workshops at an open security community. The thing about running workshops at open communties is that it is hard to predict the kinda of software/hardware people walk-in with. You\'ll have to be conscious about software dependencies and hardware requirements.

We planned to conduct a workshop on network reconnisance. We faced a bunch of challenges as part of the lab setup:

- We wanted to run couple of full blown VMs as part of labs to do remote OS detection using differences in kernel implementations so using containers is not an option.
- We wanted to run "React OS" to avoid the messy Windows licensing terms. Running React OS means, using Vagrant, containers is not an easy option.
- Audience carry laptops with various host operating systems. No native SSH client on Windows is available, so Vagrant is again not an option.
- We wanted the lab setup to be as automated as possible rather than making audience click through every step, simple Virtualbox GUI won\'t cut it.

###### VBoxManage our saviour

At this point we were not left with many options and had to turn to good old Virtualbox, that\'s when I gave a serious thought to virtualbox command-line.

- Virtualbox can run almost every *nix machines and also React OS.
- VBoxManage supports full automation of lab setup(Infact Vagrant uses VBoxManage in the backend)
- VBoxManage is available on all platforms that has Virtualbox installation.

###### Steps towards solution

**Creating the lab setup** <br>
We created a bunch of virtual machines. Few of them act as victims and one VM acts as attacker in the labs. We exported the VMs in OVA format from our Linux machine using Virtualbox GUI. At this point we had a directory with all lab VMs in OVA format.

**Importing lab setup** <br>

- A bash script for *nix and batch file for Windows were created to automate the lab setup importing using VBoxManage.
- The problem with windows is that VBoxManage is available as a command only from Virtualbox installation directory, so we had to tweak the batch file.

- The following script imports all the required OVA files and lists all the virtual machines available on the host to check if the OVAs are imported sucessfully.

```bash
#!/usr/bin/env bash
VBoxManage import "victim1.ova"
VBoxManage import "victim2.ova"
VBoxManage import "attacker.ova"
printf "\n\nList of all the VMs\n------------------------\n"
VBoxManage list vms
```

```bat
PATH=%PATH%;C:\Program Files\Oracle\VirtualBox
vboxmanage import "victim1.ova"
vboxmanage import "victim2.ova"
vboxmanage import "attacker.ova"
echo. && echo 'List of all VMs' && echo. && echo '-------------------------'
vboxmanage list vms
cmd \k
```

**Starting the labs** <br>

- A bash script for *nix and batch file for Windows were created to run the labs.
- We wanted to run the victims in the background and only display the attacker. Virtualbox headless mode runs a VM in the background.
- We exported OVAs from a Linux machine and the virtualbox host-only network adapter name on windows is not consistent with Linux so we had to use `vboxmanage modifyvm` to modify the adapter name.
- The following script run all the victims in background and displays attacker VM. This script lists all running VMs to check if everything is running.

```bash
#!/usr/bin/env bash

vboxmanage startvm "victim1" --type headless
vboxmanage startvm "victim2" --type headless
vboxmanage startvm "attacker"

printf "\n\nList of all the VMs running\n------------------------------\n"

vboxmanage list runningvms
```

```bat
PATH=%PATH%;C:\Program Files\Oracle\VirtualBox
vboxmanage modifyvm "victim1" --nic1 hostonly --hostonlyadapter1 "VirtualBox Host-Only Ethernet Adapter"
vboxmanage modifyvm "victim2" --nic1 hostonly --hostonlyadapter1 "VirtualBox Host-Only Ethernet Adapter"
vboxmanage modifyvm "attacker" --nic1 hostonly --hostonlyadapter1 "VirtualBox Host-Only Ethernet Adapter"
vboxmanage startvm "victim1" --type headless
vboxmanage startvm "victim2" --type headless
vboxmanage startvm "attacker"
echo. && echo 'List of all the running VMs' && echo. && echo '-------------------------'
vboxmanage list runningvms
cmd \k
```
**Stopping labs** <br>

- A bash script for *nix and batch file for Windows were created to gracefully shutdown the labs.
- `vboxmanage controlvm` has an option to poweroff running VMs.
- The following script shuts down all the lab VMs running and lists current running VMs to check if everything shutdown properly.

```bash
#!/usr/bin/env bash

vboxmanage controlvm "victim1" poweroff
vboxmanage controlvm "victim2" poweroff
vboxmanage controlvm "attacker" poweroff

printf "\n\nList of all the VMs running\n------------------------------\n"

vboxmanage list runningvms
```

```bat
PATH=%PATH%;C:\Program Files\Oracle\VirtualBox
vboxmanage controlvm "victim1" poweroff
vboxmanage controlvm "victim2" poweroff
vboxmanage controlvm "attacker" poweroff
echo. && echo 'List of all the running VMs' && echo. && echo '-------------------------'
vboxmanage list runningvms
cmd \k
```

###### Conclusion

VBoxManage is a pretty neat and powerful interface provided by Virtualbox. It comes handy when trying to automate your virtual environments especially when trying to distribute your environments. This article looks into one such solution to a problem, this isn\'t even tip of the iceberg, find a problem/challenge, read through VBoxManage docs and have fun!
