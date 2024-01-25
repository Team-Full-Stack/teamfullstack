---
layout: post
title: Configuring ssh on linux
date: 2024-01-24
description: Adding your ssh key to a linux server
tags: ssh linux
categories: deployment
giscus_comments: true
related_posts: false

authors:
  - name: Usman Jamil Bhatti
    url: "https://github.com/usman-jamil"
    affiliations:
      name: Allied Consultants

---

### Generate Keys

```bash
# use ssh-keygen to generate your rsa keys and save to id_test
ssh-keygen
```

### Copy the public key

```bash
# copy the contents of your public key to the clip-board (macOS)
cat id_test.pub | pbcopy
```

### Install locally

```bash
ssh-add id_test
```

### Install on remote machine

If you connect to the remote machine using VNC then do so.

```bash
# install the public key
cd ~
mkdir .ssh
cd .ssh
# create file authorized_keys
touch authorized_keys
# copy the contents of public key to the file
cat id_test.pub >> authorized_keys
```

### Connect

```bash
ssh -i id_test root@ip_or_host
```
