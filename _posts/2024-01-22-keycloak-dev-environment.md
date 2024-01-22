---
layout: post
title: Deploying a dev/test keycloak environment
date: 2024-01-22
description: Keycloak, Postgres and lets-encrypt for TLS
tags: keycloak docker lets-encrypt
categories: identity-as-a-service
giscus_comments: true
related_posts: false

authors:
  - name: Usman Jamil Bhatti
    url: "https://github.com/usman-jamil"
    affiliations:
      name: Allied Consultants

---

We are going to deploy a keycloak development server on a virtual machine. This article is based on [Johannes Reppin's](https://gitlab.desy.de/johannes.reppin/keycloak-docker-compose) implementation. The basic steps are all mentioned in the README file of this repository. However, a few key steps are worth mentioning.

Our identity server has been deployed to a virtual machine that has been hosted in GCP. For your subdomain you just need to create an `A` type record in your DNS settings that point the subdomain to the external IP of the virtual machine. Thats's it!

The initial steps are all in the [Blog](https://gitlab.desy.de/johannes.reppin/keycloak-docker-compose) and [this Plural Sight](https://app.pluralsight.com/library/courses/keycloak-getting-started/table-of-contents) course. Once you have completed the steps, clone the repo, navigate to the repository dir on your machine and run the following commands

```bash
$ docker compose pull
$ docker compose up -d
```

and Voilà! Your very own identity server is up and running. Don't you just love open-source! 
