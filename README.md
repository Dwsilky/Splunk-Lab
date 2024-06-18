# Splunk-Lab Setup
Splunk SIEM on Ubuntu Server + Windows 10 Honeypot

This repository contains the setup instructions for a home lab with an Ubuntu Server SIEM using Splunk and a vulnerable Windows VM.

## Table of Contents

- [Overview](#overview)
- [Setup](#setup)
  - [Ubuntu Server SIEM](#ubuntu-server-siem)
  - [Vulnerable Windows VM](#vulnerable-windows-vm)
- [Configuration](#configuration)
  - [Splunk Universal Forwarder](#splunk-universal-forwarder)
- [Accessing Splunk](#accessing-splunk)
- [Resources](#resources)

## Overview

This home lab setup includes an Ubuntu Server running Splunk for SIEM and a vulnerable Windows VM for security testing and monitoring. This is my first time ever interacting with Splunk, and there are no good write-ups available online about setting this up in a lab for some reason. The only write-ups I saw, the writer did not cater to a beginner audience. Hopefully my issues and frustration can help make the process easier for you.

## Network Setup and Splunk Overview

First you want to set up 2 virtual machines. I opted to use VMWare, an Ubuntu Server 24.04 VM, and a Windows 10 VM. The reason I chose these 2 is simple: I saw multiple people also running Ubuntu Server and Windows 10 to host the Splunk Indexer and Forwarder, and a simple google search stated Ubuntu server is great for the task. There are 2 parts to Splunk: the indexer and the forwarder. The indexer is the query/aggregation engine that will be hosted on Ubuntu Server. The forwarder is essentially an agent that can monitor an OS for a variety of log sources. This will live on our vulnerable Windows 10 VM. Splunk is well known for being able to aggregate and collect logs from a wide variety of sources. I am not really sure how it would work with email, but I imagine a forwarder would have to be attached to the email server OS. 

My networking skills are subpar at best so far, so I used chatGPT to determine how best to set up the network between the VMs and my host OS.

## Ubuntu Setup

First, you will load your Ubuntu ISO (most recent version) to VMWare Workstation Pro (now free). For setup, I chose 100gb memory, 4 cores, and 8gb RAM. You can go less, but I've heard that Splunk is notorious for using a lot of RAM. When your VM is powered on you will go through the basic pre-set configs until you reach the "Network Configuration" page. **IMPORTANT**: This is where we will set a static IP address for this VM so it is easier to troubleshoot issues and doesn't change throughout the lab. You need to find out the gateway IP and subnet IP of your VMWare Workstation NAT. Here is how you do it: 
1. In VMWare main interface, click edit --> Virtual Network Editor.
2. Click the Name that has "Type: NAT" then below click "NAT Settings...".
3. You need your Subnet Mask and Gateway IP from this screen for the Ubuntu setup.
4. Back on Ubuntu Network Configuration page you need to click on the option with "Type: eth" and "Edit IPv4". You need to change from DHCP to Manual IP Addressing. DHCP does not provide static addressing from what I know it is dynamic addressing so they will change depending on needs..
5. Once you choose manual you will be at a screen where you need to add your subnet. **NOTE**: this subnet has to be in CIDR notation, so take the Subnet IP from step 3 and type /24 at the end. ETC: My subnet IP was 192.168.202.0 so it became 192.168.202.0/24.
6. For the part that asks for Address, you will use the address from the Previous "Network Configuration" page next to DHCP. **Note**: this will be in CIDR notation, but when you enter the gateway it IS NOT in CIDR, so remove the /24.
7. Finally, you will use the Gateway from your VMWare Virtual Network Editor from step 3. For "Name Server": you will use 8.8.8.8 which is the IP for one of Google's public DNS servers that is widely used for speed/reliability.
8. **Important**: Make sure you write down your Ubuntu VM's IP Address as you will use it for a lot of troubleshooting (hopefully not too much) down the road.

Continue through the installation prompts as defaults until you get to the profile configuration page. Here you will set up a username and password. I make it easy to remember, but your call. **Important**: make sure you set up SSH because we love SSH for labs. 

You will be forced to restart Ubuntu once installation finished, and don't worry if reboot fails, happened to me once before mounting properly. 

Once you login, you want to make sure you properly configured the network, and your DNS is working. So do a basic ping of google.com: 

```sh
ping -c 4 google.com
```

If you aren't getting any visuals then the ping failed and you likely have to remount your ubuntu ISO and reconfigure the network settings as I specified.

## Windows Setup

I don't know if this was just a me issue or a normal issue with mounting a VM to an ISO, but for Windows, when you first load to your ISO, make sure you first click any key to continue (my load failed here for some reason when I didn't hit a key). 

Choose Windows 10 Pro. **Important**: Make sure you choose custom installation as there is no prior version of Windows loaded to disk. 

Just get yourself to the desktop of Windows, there are a lot of customization screens I am not going to walk through. Just try to avoid any features and use an offline account. 

**Turn Windows Into a Honeypot**

1. Go to search bar at bottom of screen and type Windows Security. Next go to virus and threat protection, scroll down a bit and click manage settings. TURN OFF EVERY SETTING HERE.
2. I saw from a few sources it is good to permanently disable defender from Group Policy Editor, so we will type CMD in start menu and run as administrator.
3. In CMD type:
```sh
gpedit.msc
```
4. In the Policy Editor, click Computer Configuration, Administrative Templates, Windows Components, Microsoft Defender Antivirus.
5. Double click "Turn off Microsoft Defender Antivirus", and Click Enabled, make sure you hit Apply, then Ok. 
6. From CMD as Administrator, type the following command to permanently disable Defender:
```sh
REG ADD "hklm\software\policies\microsoft\windows defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
```
7. Boot into Safe Mode to disable all Defender services
  i. Click start > msconfig
  ii. Go to the tab that says Boot and click Boot Options.
  iii. Check Safe Boot and Minimal. Apply, then hit ok. You will restart now into safe mode.
8. We are in safe mode now and will go to registry editor in search bar.
9. Under Computer\HKEY_LOCAL_MACHINE_\System\CurrentControls\Services you will look for the following final locations within this folder: 1. Sense 2. WdBoot 3. WinDefend 4. WdNisDrv 5.  WdNisSvc 6. WdFilter
10. Foreach of the previous final locations within the mentioned registry folder, you need to double click the option that says "Start" and change the value to 4.
11. Once complete exit safe mode the same way we got to it in step 7. Start > msconfig > boot tab > boot options > uncheck safe mode box > apply/ok > restart PC!
12. **Final Step for Windows VM**: We will prevent VM from going into standby mode while we leave it vulnerable by typing the following into CMD as Admin:
```sh
powercfg /change standby-timeout-ac 0
powercfg /change standby-timeout-dc 0
powercfg /change monitor-timeout-ac 0
powercfg /change monitor-timeout-dc 0
powercfg /change hibernate-timeout-ac 0
powercfg /change hibernate-timeout-dc 0
```
### Ubuntu Server SIEM

**Download and Install Splunk:**

1. Go to Splunk website and create an enterprise free trial account. I believe these trials last 60 days, so you have plenty of time to capture traffic. Once you create your account, navigate to the Splunk download and copy the wget for your use on your Ubuntu Server VM. **Note**: Make sure you choose the Linux .deb wget file. 
2. Once you use wget type ls on your ubuntu machine and you should see 2 .deb files. Whichever one is your first file, in my case it was "splunk-9.2.1-78803f08aabb-linux-2.6-amd64.deb", you will type the following command with it to extract the file in order to use Splunk. **Note**: the following command CONTAINS WHAT MY FILE LOOKED LIKE, so if your file has a different name then change the command accordingly.
```sh
sudo dpkg -i splunk-9.2.1-78803f08aabb-linux-2.6-amd64.deb
```
**Start Splunk on Ubuntu:**

Use the following commands to first start splunk and accept license, then to enable splunk to run on boot. *Note: If these don't work then you likely didn't install it to the directory that is mentioned. Make sure you are using the command for the directory that Splunk is in. **Important**: When you accept the license it will have you set up a user and password. Make sure you write this down or put in notepad, because you will use it in the web interface for Splunk to log in.

```sh
sudo /opt/splunk/bin/splunk start --accept-license
sudo /opt/splunk/bin/splunk enable boot-start
```

**Access Splunk Web GUI on Windows VM**

So we are now getting to the part where I started encountering a lot of issues.

1. From the Windows VM go to the internet and type http:"linux-vm-ipaddress":8000 
2. If you don't remember your Ubuntu Server ip address you can type the linux command:
```sh
ip addr show
```
There should only be 2 options, the loopback or lo: and the network interface we set up, which should look like `eth0`, `ens33`, or `enp0s3` etc..
You are looking for the latter option, and you will want to look under `inet` for the correct IP.

3. Once you enter the ip in the Windows VM:8000 for the default port of 8000, you should see the web page for Splunk. Log in with the credentials you set up when you initialized Splunk on Ubuntu.
4. VIOLA, you are in and you completed the setup of the Indexer, good job pal.

**Splunk Universal Forwarder**

Now we will install the Universal Forwarder on our Windows VM to capture Windows Event logs. 

