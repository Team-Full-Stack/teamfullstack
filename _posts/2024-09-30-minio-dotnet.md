---
layout: distill
title: Integrating MinIO with .NET Applications
description: A guide to using MinIO as an object storage solution in .NET applications.
tags: MinIO, .NET, Object Storage
giscus_comments: true
date: 2024-09-30
featured: false

authors:
  - name: Usman Jamil Bhatti
    url: "https://github.com/usman-jamil"
    affiliations:
      name: Allied Consultants
  - name: Sheikh Luqman
    url: "https://github.com/luqmant51"
    affiliations:
      name: Allied Consultants
  - name: Zuhaib Sohail
    url: "https://github.com/ZuhaibSohail163"
    affiliations:
      name: Allied Consultants

bibliography: 2018-12-22-krazzy.bib

toc:
  - name: Introduction
  - name: Setting Up MinIO
  - name: Implementing MinIO Commands in .NET
    subsections:
      - name: Create Bucket
      - name: Delete Bucket
      - name: List Buckets
      - name: File Operations
        subsections:
          - name: Upload File
          - name: Download File
          - name: Delete File
          - name: List Files
  - name: Conclusion

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

## Introduction

MinIO is a high-performance, S3-compatible object storage service that is perfect for cloud-native applications. In this guide, we will explore how to integrate MinIO with .NET applications, providing you with the ability to create, delete, and manage buckets and files seamlessly.

## Setting Up MinIO

Before diving into code, ensure that you have MinIO installed and running. You can use the following Dockerfile to set up MinIO with a client for managing buckets and policies:

### Dockerfile

```dockerfile
# Use the official MinIO image
FROM minio/minio:latest

# Set the MinIO data directory (this is where your data will be stored)
ENV MINIO_VOLUMES="/data"

# Set MinIO access and secret keys
ENV MINIO_ROOT_USER=<your-access-key> \
    MINIO_ROOT_PASSWORD=<your-secret-key>

# Expose MinIO API (default port 9000) and console (port 9001)
EXPOSE 9000 9001

# Download MinIO Client (mc) for managing buckets and policies using curl
RUN curl -O https://dl.min.io/client/mc/release/linux-amd64/mc \
    && chmod +x mc \
    && mv mc /usr/local/bin/

# Copy the script to create buckets and set policies
COPY create-buckets.sh /usr/local/bin/create-buckets.sh
RUN chmod +x /usr/local/bin/create-buckets.sh

# Use bash to handle the script execution
ENTRYPOINT ["bash", "-c", "/usr/local/bin/create-buckets.sh & minio server /data --console-address :9001"]
```

### docker-compose.yml
```dockerfile
version: '3.8'

services:
minio:
build:
context: .
dockerfile: Dockerfile
ports:
- "9000:9000"      # MinIO API
- "9001:9001"      # MinIO Console
environment:
MINIO_ROOT_USER: <your-access-key>
MINIO_ROOT_PASSWORD: <your-secret-key>
volumes:
      - minio-data:/data  # Persistent data storage

volumes:
  minio-data:
```

### Command for
```bash
docker-compose up --build
```
For the complete code and implementation, you can visit the public repository: [MinioServerConsoleApp](https://github.com/Luqmant51/MinioServerConsoleApp).