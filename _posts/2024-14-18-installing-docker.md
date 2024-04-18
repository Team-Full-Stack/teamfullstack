---
layout: distill
title: Install Docker
description: Easy step to install docker
tags: install Docker
giscus_comments: true
date: 2024-04-18
featured: false

authors:
  - name: Usman Jamil Bhatti
    url: "https://github.com/usman-jamil"
    affiliations:
      name: Allied Consultants
  - name:  Sheikh Luqman
    url: "https://en.wikipedia.org/wiki/Boris_Podolsky"
    affiliations:
      name: Allied Consultants
  - name: Zuhaib Sohail
    url: "https://github.com/ZuhaibSohail163"
    affiliations:
      name: Allied Consultants

_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }

---

How to install Docker

## Installing Docker

Before you start setting up your project, you need to install Docker on your machine if it isn't already installed. Docker allows you to create, deploy, and run applications by using containers.

### For Windows

1. Download Docker Desktop from the [official Docker website](https://www.docker.com/products/docker-desktop).
2. Run the installer and follow the on-screen instructions.
3. Ensure that the "Windows Subsystem for Linux" (WSL2) feature is enabled on your machine.
4. After installation, launch Docker Desktop and sign in or create a Docker account if prompted.

### For macOS

1. Download Docker Desktop for Mac from the [official Docker website](https://www.docker.com/products/docker-desktop).
2. Open the downloaded `.dmg` file and drag the Docker icon to your Applications folder.
3. Open Docker from your Applications to complete the installation.
4. Docker may request your password to install a helper tool; provide your password to continue.

### For Linux

The installation commands can vary depending on your Linux distribution. Here's how you can install Docker on Ubuntu:

1. Update your existing list of packages:
```bash
  $ sudo apt update

  $ sudo apt install apt-transport-https ca-certificates curl software-properties-common

  $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

  $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

  $ sudo apt update

  $ apt-cache policy docker-ce

  $ sudo apt install docker-ce

docker run hello-world
```
2. Or Using this link just copy and past
```bash
curl https://get.docker.com/ | bash
```
or
```bash
curl -fsSL https://get.docker.com/ -o get-docker.sh
```