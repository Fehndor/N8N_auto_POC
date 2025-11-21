# Stable n8n Deployment on AWS EC2 — Step-by-Step Setup Guide (Version 1.69.2)

![badge](https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws)
![badge](https://img.shields.io/badge/Docker-Ready-blue?logo=docker)
![badge](https://img.shields.io/badge/n8n-1.69.2-green?logo=n8n)
![badge](https://img.shields.io/badge/License-MIT-yellow)

---

# 📚 Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [1. Launch the EC2 Instance](#1-launch-the-ec2-instance)
- [2. Connect to the Server](#2-connect-to-the-server)
- [3. Prepare Folders for n8n](#3-prepare-folders-for-n8n)
- [4. Create docker-compose.yml](#4-create-docker-composeyml)
- [5. Start n8n](#5-start-n8n)
- [6. First-Time Setup](#6-first-time-setup)
- [7. Why We Use Version 1.69.2](#7-why-we-use-version-1692)
- [Next Steps](#next-steps)
- [License](#license)

---

# Overview

This guide walks you through deploying **n8n on AWS EC2** using the **stable version 1.69.2**.  
We intentionally use this version because newer n8n releases introduce a frontend bug that causes the UI to crash when deployed without HTTPS.

This setup is ideal for:

- testing  
- development  
- Jira integrations  
- internal/home-lab automations  

Screenshots can be added later.

---

# Prerequisites

✔ AWS account  
✔ SSH client  
✔ Basic command-line familiarity  
✔ Port 5678 accessible from your IP  

---

# 1. Launch the EC2 Instance

1. Log into AWS and navigate to **EC2 → Launch Instance**.
2. Choose:
   - **Name:** `n8n-server`
   - **AMI:** Ubuntu Server 22.04 LTS
   - **Architecture:** x86_64
   - **Instance type:** `t2.micro` (free tier)
3. Create a **new RSA key pair** and download the `.pem` file.
4. Configure security group:
   - Allow **SSH (22)** from your IP
   - Allow **HTTP (5678)** from your IP
5. Launch the instance.

---

# 2. Connect to the Server

SSH into the instance:

```bash
ssh -i path/to/key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Update the OS:

```bash
sudo apt update && sudo apt upgrade -y
```

Install Docker + Docker Compose:

```bash
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker --now
```

---

# 3. Prepare Folders for n8n

```bash
sudo mkdir -p /var/n8n
sudo chown -R 1000:1000 /var/n8n
sudo chmod -R 755 /var/n8n
```

This ensures correct permissions for n8n volumes.

---

# 4. Create docker-compose.yml

Inside your home directory:

```bash
nano docker-compose.yml
```

Paste the following:

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:1.69.2
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=test1234
      - N8N_SECURE_COOKIE=false
      - N8N_DIAGNOSTICS_ENABLED=false
    volumes:
      - /var/n8n:/home/node/.n8n
```

Save and exit.

---

# 5. Start n8n

Run:

```bash
docker-compose up -d
```

Check logs:

```bash
docker logs n8n_n8n_1
```

n8n is now available at:

```
http://YOUR_EC2_PUBLIC_IP:5678
```

---

# 6. First-Time Setup

1. Open n8n in your browser.
2. Complete the **owner account** creation.
3. You are now inside the n8n editor.

No HTTPS required thanks to:

```
N8N_SECURE_COOKIE=false
```

---

# 7. Why We Use Version 1.69.2

Later versions of n8n (1.120.x and newer):

- introduce a redesigned “Security Core”
- assume SSO/LDAP structures exist
- break when running on plain HTTP over an IP
- often cause the UI to crash during initialization

Version **1.69.2** is:

- more stable  
- simpler  
- compatible with plain HTTP  
- ideal for testing environments  

---

# Next Steps

You may now:

- integrate **Jira webhooks**
- build automation workflows
- add HTTPS later using Traefik or Caddy
- create backups of `/var/n8n`

If you'd like, I can generate a matching **Jira → n8n integration guide**.

---

# License

MIT License. Feel free to use and modify this guide.

