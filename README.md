# Buildbot CI/CD and Installation Guide

This document provides instructions for setting up Buildbot as a Continuous Integration (CI) and Continuous Deployment (CD) system, as well as a detailed guide on installing Buildbot on Ubuntu 16.04.

---

## Table of Contents
1. Introduction
2. Requirements
3. Buildbot CI/CD Setup
4. Buildbot Installation on Ubuntu 16.04
5. Contributing
6. License

---

## 1. Introduction

Buildbot is a Python-based system that automates software build, test, and release processes. It uses Twisted for asynchronous communication between a buildmaster and one or more workers. This guide includes:
- A modern CI/CD setup using Buildbot.
- An installation guide for Buildbot on Ubuntu 16.04.

---

## 2. Requirements

- Python 3.x
- pip and pipx
- An Ubuntu 16.04 server (for the installation guide) or a modern server for CI/CD purposes
- Buildbot and Buildbot Worker (installed via pipx)

---

## 3. Buildbot CI/CD Setup

### 3.1 Install Buildbot Master

Install Buildbot with the bundle using pipx:

    pipx install 'buildbot[bundle]'

### 3.2 Create and Configure the Master

1. Create the master directory:

       mkdir -p /home/buildbot/master
       buildbot create-master /home/buildbot/master

2. Copy the sample configuration:

       cp /home/buildbot/master/master.cfg.sample /home/buildbot/master/master.cfg

3. Edit the configuration file:

       nano /home/buildbot/master/master.cfg

   Change the `buildbotURL` to your desired URL. For example:

       c['buildbotURL'] = "https://build.poecdn.cloud/"

4. Restart the master to apply changes:

       buildbot restart /home/buildbot/master
       tail -f /home/buildbot/master/twistd.log

   Confirm that the log shows "BuildMaster is running".

### 3.3 Configure the Buildbot Worker

1. Create the worker directory:

       mkdir -p ~/worker

2. Create the worker instance (replace worker_name and password as needed):

       buildbot-worker create-worker ~/worker worker_name password

3. Update the host information:

       echo $(hostname) > ~/worker/info/host
       cat ~/worker/info/host

4. Start the worker:

       buildbot-worker start ~/worker

   If an instance is already running, use:

       buildbot-worker status ~/worker
       buildbot-worker stop ~/worker

   Then start the worker again.

### 3.4 Triggering a Build

There are two methods:

#### A. Web Interface

- Open your browser and navigate to:  
  https://build.poecdn.cloud/
- Select your builder (e.g., "my-builder").
- Click **Force Build** to trigger a build.

#### B. Command Line

Trigger a build using:

    buildbot sendchange --master https://build.poecdn.cloud/ --branch master --revision HEAD --who "Your Name" --comments "Manually triggering build" my-builder

### 3.5 Monitoring and Logs

- Check master status:

       buildbot status /home/buildbot/master

- Check worker status:

       buildbot-worker status ~/worker

- Monitor master logs:

       tail -f /home/buildbot/master/twistd.log

- Monitor worker logs:

       tail -f ~/worker/twistd.log

---

## 4. Buildbot Installation on Ubuntu 16.04

### 4.1 Introduction

This section details the installation and configuration of Buildbot on Ubuntu 16.04, setting up both the master and worker on the same machine.

### 4.2 Prerequisites

- An Ubuntu 16.04 server with at least 1 GB of RAM.
- A non-root sudo user.
- UFW firewall configured to allow SSH.

### 4.3 Installing Buildbot

1. Update package list:

       sudo apt-get update

2. Install pip:

       sudo apt-get install python-pip

3. Install the Buildbot bundle:

       sudo -H pip install 'buildbot[bundle]'

4. (Optional) Upgrade pip:

       sudo -H pip install --upgrade pip

5. Verify the installation:

       buildbot --version

   Expected output:

       Buildbot version: 1.0.0
       Twisted version: 17.9.0

6. Open port 8010 for the Buildbot web interface:

       sudo ufw allow 8010

7. Create a dedicated Buildbot system user and group:

       sudo addgroup --system buildbot
       sudo adduser buildbot --system --ingroup buildbot --shell /bin/bash

8. Switch to the Buildbot user:

       sudo --login --user buildbot

### 4.4 Configuring the Master

1. Create the master:

       buildbot create-master ~/master

   This creates `/home/buildbot/master/master.cfg.sample` and a SQLite database.

2. Copy the sample configuration:

       cp ~/master/master.cfg.sample ~/master/master.cfg

3. Edit the configuration file:

       nano ~/master/master.cfg

   Change `buildbotURL` to your server's IP or domain:

       c['buildbotURL'] = "http://IP_or_site_domain:8010/"

   Optionally set:

       c['buildbotNetUsageData'] = None
   or
       c['buildbotNetUsageData'] = 'basic'

4. Check the configuration and start the master:

       buildbot checkconfig ~/master
       buildbot start ~/master

5. Access the web interface by opening:

       http://IP_or_site_domain:8010/

### 4.5 Configuring a Worker

1. Create the worker:

       buildbot-worker create-worker ~/worker localhost example-worker pass

2. Edit the worker information files:

   - Update the admin file:

         nano ~/worker/info/admin

     Replace the sample text with the appropriate administrator information.

   - Update the host file with system details:

         nano ~/worker/info/host

     For example:

         Ubuntu 16.04.2 Server - Buildbot version: 1.0.0 - Twisted version: 17.1.0

3. Start the worker:

       buildbot-worker start ~/worker

### 4.6 Running a Test Build

- In the web interface, navigate to "Workers" to verify your worker details.
- Click on the default builder (e.g., "runtests") and press **Force Build**.
- The build should complete successfully; detailed logs can be viewed by clicking on each step.

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

This ensures that administrative functions require login.

---

## 5. Contributing

For internal documentation, please direct any changes or updates to the relevant team members.

---

## 6. License

This documentation is for internal use.

