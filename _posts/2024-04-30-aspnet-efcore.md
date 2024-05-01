---
layout: distill
title: ASP.net 8.0 and Entity Core Framework with postgres sql and docker compose
description: ASP.net 8.0 and Entity Core Framework with postgres sql and docker compose
tags: Docker-compose With dotnet and next
giscus_comments: true
date: 2024-04-18
featured: false

authors:
  - name: Usman Jamil Bhatti
    url: "https://github.com/usman-jamil"
    affiliations:
      name: Allied Consultants
  - name:  Sheikh Luqman
    url: "https://github.com/luqmant51"
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

# ASP.net 8.0 and Entity Core Framework with Postgres SQL and docker-compose

Create Web API Project 

Here is the reference of the project public repo in GitHub [Source Code](https://github.com/usman-jamil/EFCore) of the given example project asp-next-nginx-compose.


This will generate a `Dockerfile` in your project directory, which might look something like this:
```Dockerfile
version: '3.8'
services:
  postgres:
    container_name: container-pg
    image: postgres
    hostname: localhost
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: test_db
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres-data:
```

In the terminal navigate to the root folder of the project

## Run Postgres for this sample
```shell
docker compose -f docker-compose-postgres.yml up -d
```

### Here is the command to run migrations 

This command is used to create a migration:
```bash
dotnet ef migrations add --project EFCore.Playground.Infrastructure/EFCore.Playground.Infrastructure.csproj --startup-project EFCore.Playground/EFCore.Playground.csproj --context EFCore.Playground.Infrastructure.ApplicationDbContext Initial --output-dir Migrations
```
This command is used to create a database:
```bash
dotnet ef database update --project EFCore.Playground.Infrastructure/EFCore.Playground.Infrastructure.csproj --startup-project EFCore.Playground/EFCore.Playground.csproj --context EFCore.Playground.Infrastructure.ApplicationDbContext
```

### Building All the Images

To build all the images specified in your `docker-compose.yml` file, use the following command from the root directory of your project (where your `docker-compose.yml` is located):

```bash
docker-compose build
```
This command will build all the Docker images for the services defined in your `docker-compose.yml`.

### Running the Application

To start all the services defined in your `docker-compose.yml` file, use the following command:
```bash
docker-compose up
```

This will start all the containers. If you want to run them in detached mode (in the background), you can add the `-d` flag:
```bash
docker-compose up -d
```

### Stopping the Application

To stop the running containers without removing them, you can use:
```bash
docker-compose stop
```

### Removing All the Images

To remove all images created by the build process, you must first stop and remove the containers, networks, volumes, and images associated with your `docker-compose.yml` file. Here are the commands to do that:

1. Stop the containers if they are running:
```bash
docker-compose down
```

2. Remove the images:
```bash
docker-compose down --rmi all
```

Alternatively, if you want to remove all Docker images (including those not associated with this project), you can use:
```bash
docker image prune -a
```

This command will remove all images not associated with a running container, or:
```bash
docker rmi $(docker images -a -q)
```

This command forces the removal of all images by getting the IDs of all images and passing them to `docker rmi`. 
