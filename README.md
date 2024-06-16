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

   ```sh
   wget -O splunk-latest.deb "https://download.splunk.com/products/splunk/releases/8.2.4/linux/splunk-8.2.4-ddff1c41e5cf-linux-2.6-amd64.deb"
   sudo dpkg -i splunk-latest.deb
