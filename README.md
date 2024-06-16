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

This home lab setup includes an Ubuntu Server running Splunk for SIEM and a vulnerable Windows VM for security testing and monitoring.

## Setup

### Ubuntu Server SIEM

1. **Download and Install Splunk:**

Go to Splunk website and create an enterprise free trial account. I believe these trials last 60 days, so you have plenty of time to capture traffic. Once you create your account, navigate to the Splunk download and copy the wget for your use on your Ubuntu Server VM.

   ```sh
   wget -O splunk-latest.deb "https://download.splunk.com/products/splunk/releases/8.2.4/linux/splunk-8.2.4-ddff1c41e5cf-linux-2.6-amd64.deb"
   sudo dpkg -i splunk-latest.deb
   ```sh

2. **Start Splunk:**

Use the following commands to first start splunk and accept license, then to enable splunk to run on boot. *Note: If these don't work then you likely didn't install it to the directory that is mentioned. Make sure you are using the command for the directory that Splunk is in.

  ```sh
  sudo /opt/splunk/bin/splunk start --accept-license
  sudo /opt/splunk/bin/splunk enable boot-start

  
