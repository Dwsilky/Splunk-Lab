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

First, you will load your Ubuntu ISO (most recent version) to VMWare Workstation Pro (now free). For setup, I chose 100gb memory, 4 cores, and 8gb RAM. You can go less, but I've heard that Splunk is notorious for using a lot of RAM. When your VM is powered on you will go through the basic pre-set configs until you reach the "Network Configuration" page. **IMPORTANT**: This is where we will set a static IP address for this VM so it is easier to troubleshoot issues and doesn't change throughout the lab. You need to find out the gateway IP of your VMWare Workstation NAT. Here is how you do it: 
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

Once you login, you want to make sure you properly configured the network, so do a basic ping of google.com 

```sh
ping -c 4 google.com
```

### Ubuntu Server SIEM

1. **Download and Install Splunk:**

Go to Splunk website and create an enterprise free trial account. I believe these trials last 60 days, so you have plenty of time to capture traffic. Once you create your account, navigate to the Splunk download and copy the wget for your use on your Ubuntu Server VM.

   ```sh
   wget -O splunk-latest.deb "https://download.splunk.com/products/splunk/releases/8.2.4/linux/splunk-8.2.4-ddff1c41e5cf-linux-2.6-amd64.deb"
   sudo dpkg -i splunk-latest.deb
   ```

2. **Start Splunk:**

Use the following commands to first start splunk and accept license, then to enable splunk to run on boot. *Note: If these don't work then you likely didn't install it to the directory that is mentioned. Make sure you are using the command for the directory that Splunk is in.

  ```sh
  sudo /opt/splunk/bin/splunk start --accept-license
  sudo /opt/splunk/bin/splunk enable boot-start

  
