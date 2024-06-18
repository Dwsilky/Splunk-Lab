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

First you want to set up 2 virtual machines. I opted to use VirtualBox, an Ubuntu Server 24.04 VM, and a Windows 10 VM. The reason I chose these 2 is simple: I saw multiple people also running Ubuntu Server and Windows 10 to host the Splunk Indexer and Forwarder, and a simple google search stated Ubuntu server is great for the task. There are 2 parts to Splunk: the indexer and the forwarder. The indexer is the query/aggregation engine that will be hosted on Ubuntu Server. The forwarder is essentially an agent that can monitor an OS for a variety of log sources. This will live on our vulnerable Windows 10 VM. Splunk is well known for being able to aggregate and collect logs from a wide variety of sources. I am not really sure how it would work with email, but I imagine a forwarder would have to be attached to the email server OS. 

My networking skills are subpar at best so far, so I used chatGPT to determine how best to set up the network between the VMs and my host OS. I set up 2 network adaptors (NICs) on EACH VM. The first network adaptor for each VM will be NAT, and the second will be Internal Network (intnet on Virtualbox). NAT allows internet access for the VMs and Internal Network allows interhost communication between the VMs. 

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

  
