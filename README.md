# Buildbot CI/CD and Installation Guide (Debian/CentOS)

This document provides internal instructions for setting up Buildbot as a Continuous Integration (CI) and Continuous Deployment (CD) system, as well as a detailed guide for installing Buildbot on Debian and CentOS systems. It also includes steps to configure Nginx as a reverse proxy with HTTPS using Certbot.

---

## Table of Contents
1. Introduction
2. Requirements
3. Buildbot CI/CD Setup
4. Buildbot Installation on Debian and CentOS
   4.1 Introduction
   4.2 Prerequisites
   4.3 Install Buildbot
   4.4 Configure the Master
   4.5 Configure the Worker
   4.6 Running a Test Build
   4.7 Securing the Web Interface
   4.8 Configure Nginx as a Reverse Proxy with Certbot
5. Contributing
6. License

---

## 1. Introduction

Buildbot is a Python-based system that automates software build, test, and deployment processes. It uses the Twisted library to handle asynchronous communication between a buildmaster and one or more workers. This document covers:
- A modern CI/CD setup using Buildbot.
- An installation guide for Buildbot on Debian and CentOS.
- Steps to secure the Buildbot web interface using Nginx and Certbot.

---

## 2. Requirements

- Python 3.x
- pip and pipx
- A server running Debian or CentOS
- Buildbot and Buildbot Worker (installed via pipx)

---

## 3. Buildbot CI/CD Setup

### 3.1 Install Buildbot Master

Install Buildbot with the bundle using pipx:

    pipx install 'buildbot[bundle]'

### 3.2 Create and Configure the Master

1. **Create the master directory:**

       mkdir -p /home/buildbot/master
       buildbot create-master /home/buildbot/master

2. **Copy the sample configuration:**

       cp /home/buildbot/master/master.cfg.sample /home/buildbot/master/master.cfg

3. **Edit the configuration file:**

       nano /home/buildbot/master/master.cfg

   Change the `buildbotURL` line to point to your desired URL, for example:

       c['buildbotURL'] = "https://build.poecdn.cloud/"

4. **Restart the master to apply changes:**

       buildbot restart /home/buildbot/master
       tail -f /home/buildbot/master/twistd.log

   Verify that the log shows "BuildMaster is running".

### 3.3 Configure the Buildbot Worker

1. **Create the worker directory:**

       mkdir -p ~/worker

2. **Create the worker instance (replace worker_name and password as needed):**

       buildbot-worker create-worker ~/worker worker_name password

3. **Update host information:**

       echo $(hostname) > ~/worker/info/host
       cat ~/worker/info/host

4. **Start the worker:**

       buildbot-worker start ~/worker

   If an instance is already running, check its status:

       buildbot-worker status ~/worker
       buildbot-worker stop ~/worker

   Then start the worker again.

### 3.4 Triggering a Build

There are two methods:

#### A. Web Interface

- Open your browser and navigate to:  
  https://build.poecdn.cloud/
- Select your builder (e.g., "my-builder").
- Click **Force Build** to trigger a build manually.

#### B. Command Line

Trigger a build using:

    buildbot sendchange --master https://build.poecdn.cloud/ --branch master --revision HEAD --who "Your Name" --comments "Manually triggering build" my-builder

### 3.5 Monitoring and Logs

- **Master status:**

       buildbot status /home/buildbot/master

- **Worker status:**

       buildbot-worker status ~/worker

- **Monitor master logs:**

       tail -f /home/buildbot/master/twistd.log

- **Monitor worker logs:**

       tail -f ~/worker/twistd.log

---

## 4. Buildbot Installation on Debian and CentOS

### 4.1 Introduction

This section details the installation and configuration of Buildbot on Debian and CentOS systems, setting up both the master and worker on the same machine.

### 4.2 Prerequisites

- A server running Debian or CentOS with at least 1 GB of RAM.
- A non-root sudo user.
- A firewall configured to allow SSH and port 8010 (for the Buildbot web interface).

### 4.3 Install Buildbot

1. **Update package list:**

       sudo apt-get update   # For Debian
       sudo yum update       # For CentOS

2. **Install pip:**

       sudo apt-get install python-pip   # For Debian
       sudo yum install python-pip       # For CentOS

3. **Install the Buildbot bundle:**

       sudo -H pip install 'buildbot[bundle]'

4. **(Optional) Upgrade pip:**

       sudo -H pip install --upgrade pip

5. **Verify the installation:**

       buildbot --version

   Expected output similar to:

       Buildbot version: X.X.X
       Twisted version: X.X.X

6. **Configure the firewall to allow port 8010:**

       sudo ufw allow 8010   # For Debian (if using ufw)
       # For CentOS, configure firewalld or iptables to allow port 8010.

7. **Create a dedicated Buildbot system user and group:**

   For Debian:

       sudo addgroup --system buildbot
       sudo adduser buildbot --system --ingroup buildbot --shell /bin/bash

   For CentOS:

       sudo groupadd -r buildbot
       sudo useradd -r -g buildbot -s /bin/bash buildbot

8. **Switch to the Buildbot user:**

       sudo --login --user buildbot

### 4.4 Configure the Master

1. **Create the master:**

       buildbot create-master ~/master

   This creates the sample configuration and a SQLite database.

2. **Copy the sample configuration:**

       cp ~/master/master.cfg.sample ~/master/master.cfg

3. **Edit the configuration file:**

       nano ~/master/master.cfg

   Change the `buildbotURL` to your server's IP or domain:

       c['buildbotURL'] = "http://IP_or_site_domain:8010/"

   Optionally, set:

       c['buildbotNetUsageData'] = None
   or
       c['buildbotNetUsageData'] = 'basic'

4. **Check the configuration and start the master:**

       buildbot checkconfig ~/master
       buildbot start ~/master

5. **Access the web interface by navigating to:**

       http://IP_or_site_domain:8010/

### 4.5 Configure the Worker

1. **Create the worker:**

       buildbot-worker create-worker ~/worker localhost example-worker pass

2. **Edit the worker information files:**

   - **Update the admin file:**

         nano ~/worker/info/admin

     Replace the sample text with the appropriate administrator information.

   - **Update the host file** (use generic system information without specifying the machine model):

         nano ~/worker/info/host

     For example:

         Debian/CentOS Server - Buildbot version: X.X.X - Twisted version: X.X.X

3. **Start the worker:**

       buildbot-worker start ~/worker

### 4.6 Running a Test Build

- In the web interface, navigate to "Workers" to verify your worker details.
- Click on the default builder (e.g., "runtests") and press **Force Build**.
- The build should complete successfully; detailed logs can be viewed for each step.

### 4.7 Securing the Web Interface

To secure the web interface, add the following lines at the bottom of `~/master/master.cfg` (modify as needed):

    c['www']['authz'] = util.Authz(
           allowRules = [ util.AnyEndpointMatcher(role="admins") ],
           roleMatchers = [ util.RolesFromUsername(roles=['admins'], usernames=['admin']) ]
    )
    c['www']['auth'] = util.UserPasswordAuth({'admin': 'password'})

Check the configuration and restart the master:

    buildbot checkconfig ~/master
    buildbot restart ~/master

This ensures that administrative functions require authentication.

### 4.8 Configure Nginx as a Reverse Proxy with Certbot

Using Nginx and Certbot helps secure the Buildbot web interface with HTTPS.

#### 4.8.1 Install Nginx and Certbot

Run the following commands:

    sudo apt update
    sudo apt install nginx -y
    sudo apt install certbot python3-certbot-nginx -y

#### 4.8.2 Create an Nginx Site Configuration

Edit a new configuration file for your domain (replace "build.poecdn.cloud" with your domain):

    sudo nano /etc/nginx/sites-available/build.poecdn.cloud

Insert the following content:

```nginx
server {
    listen 80;
    server_name build.poecdn.cloud;
    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name build.poecdn.cloud;

    ssl_certificate /etc/letsencrypt/live/build.poecdn.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/build.poecdn.cloud/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://127.0.0.1:8010;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
