---
layout: post
title: Deploying keycloak with a purchased SSL
date: 2024-04-22
description: Keycloak, Postgres, Nginx and positive SSL
tags: keycloak docker-compose
categories: identity-as-a-service, ssl
giscus_comments: true
related_posts: false

authors:
  - name: Usman Jamil Bhatti
    url: "https://github.com/usman-jamil"
    affiliations:
      name: Allied Consultants
  - name:  Sheikh Luqman
    url: "https://github.com/luqmant51/"
    affiliations:
      name: Allied Consultants
  - name: Zuhaib Sohail
    url: "https://github.com/ZuhaibSohail163"
    affiliations:
      name: Allied Consultants
---

# Introduction
We can deploy a working keycloak instance on a VM (with a static IP address) and a domain with an `A type custom record` pointing to this IP address. 
For this example we will consider the domain `sso.example.com` that points to the static IP address of your VM.

## Prepare Certificates
Let just assemble the files that we acquired while buying the SSL certificate. Lets first merge our intermediate and root Certificates into a single one:

```bash
cat identity_example_com_domain.crt intermediate.crt root.crt >> sso.example.com.crt
```

If you received the intermediate Certificates in one bundle file then you can do something like

```bash
cat identity_example_com_domain.crt identity_example_com_domain.ca-bundle >> sso.example.com.crt
```

Now you should have 2 files:
 - sso.example.com.crt (your public key)
 - sso.example.com.key (your private key)

## Directory Structure
The directory structure should be something like the following:

```shell
keycloak
├── docker-compose.yml
├── keycloak.conf
├── nginx
│   └── nginx.conf
└── ssl
```

### nginx.conf
You might need to adjust this file based on your needs. But it should work pretty much as-is.
This particular confguration will do SSL termination for you. 
Notice the line `proxy_pass         http://sso:8080`, where `sso` is the name of the keycloak docker service and keycloak runs of 8080 for http traffic.

```conf
worker_processes 1;

events { worker_connections 1024; }

http {
    sendfile on;
    large_client_header_buffers 4 32k;

    server {
        listen 80;
        server_name sso.example.com;

        location / {
            return 301 https://$host$request_uri;
        }
    }

    server {
        listen 443 ssl;
        server_name sso.example.com;

        ssl_certificate /etc/ssl/certs/sso.example.com.crt;
        ssl_certificate_key /etc/ssl/private/sso.example.com.key;

        location / {
            proxy_pass         http://sso:8080;
            proxy_redirect     off;
            proxy_http_version 1.1;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_buffer_size           128k;
            proxy_buffers               4 256k;
            proxy_busy_buffers_size     256k;
        }
    }
}
```

### keycloak.conf
We map this file as a volume to our keycloak instance so it knows information such as which database provider to use etc.

```conf
proxy=edge
db=postgres
db-url-host=postgres
db-user=keycloak
db-password=
db-database=keycloak
db-schema=public
hostname-strict=false
http-enabled=true
```

### docker-compose.yml
There are a few important things to notice:
 - How our nginx.conf is passed to the nginx instance, using volumes
 - How our keycloak.conf is passed to the keycloak instance, using volumes
 - How the certificates are being passed to the nginx instance which are also mentioned by location in nginx.conf

```yml
version: "3.7"

services:    
  sso:
    image: quay.io/keycloak/keycloak:24.0
    container_name: "keycloak"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./keycloak.conf:/opt/keycloak/conf/keycloak.conf
    command:
      - start
    environment:
      - KEYCLOAK_ADMIN=
      - KEYCLOAK_ADMIN_PASSWORD=
      - PROXY_ADDRESS_FORWARDING=true
      - VIRTUAL_HOST=sso.example.com
      - VIRTUAL_PORT=8080
      - LETSENCRYPT_HOST=sso.example.com
      - KC_DB_PASSWORD=p0HfTu3dnW6G3
    networks:
      - internal
    depends_on:
      - database

  database:
    image: postgres:15
    container_name: "postgres"
    environment:
      - POSTGRES_USER=keycloak
      - POSTGRES_DATABASE=keycloak
      - POSTGRES_PASSWORD=p0HfTu3dnW6G3
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - internal

  reverseproxy:
    image: nginx:1.23
    container_name: "reverseproxy"
    ports:
      - "443:443" 
      - "80:80" 
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl/sso.example.com.crt:/etc/ssl/certs/sso.example.com.crt
      - ./ssl/sso.example.com.key:/etc/ssl/private/sso.example.com.key
    networks:
      - internal

networks:
  internal:
    driver: bridge
    driver_opts:
      # Openstack spezifisch, kann auf 1500 gelassen werden wenn ihr auf
      # Bare Metal lauft. 
      com.docker.network.driver.mtu: 1450

volumes:
  postgres_data:
```

# Lets run it
Lets cross our fingers and run the infamous command and hope it works. Make sure you are inside the keycloak folder when you run this

```shell
docker compose up -d
```

If we did everything right, a running keycloak instance should be available at: https://sso.example.com and the `http` equivalent should auto redirect to `https`
