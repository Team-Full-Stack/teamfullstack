---
layout: post
title: Deploying keycloak on bare metal with https
date: 2024-01-22 12:00:00-0500
description: Keycloak, Postgres and lets-encrypt for TLS
tags: keycloak docker lets-encrypt
categories: identity
giscus_comments: true
related_posts: true
---
We are going to deploy a keycloak development server on a virtual machine. This article is based on [Johannes Reppin's](https://gitlab.desy.de/johannes.reppin/keycloak-docker-compose) implementation. The basic steps are all mentioned in the README file of the mentioned repository. However, a few key steps are worth mentioning.

Our identity has been deployed to a virtual machine that has been hosted in GCP. 
You just need to create an `A` type record in your DNS settings that point the subdomain to the external IP of the virtual machine. Thats's it.

The rest that you need to do is pretty much mentioned in the mentioned repository. Once you have completed those steps, navigate to your repository dir on your machine and run the following commands

```bash
$ docker compose pull
$ docker compose up -d
```

and Voilà! Your very own identity server is up and running. Don't you just love open-source! 
