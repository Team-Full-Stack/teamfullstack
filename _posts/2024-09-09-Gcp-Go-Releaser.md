---
layout: distill
title: Automating Go-Releaser with GCP Cloud Build for Artifact Storage
description: A step-by-step guide to running Go-Releaser within GCP Cloud Build and uploading generated artifacts to Google Cloud Storage, organized by version.
tags: Go-Releaser, GCP Cloud Build, Google Cloud Storage, CI/CD
discus_comments: true
date: 2024-09-09
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

bibliography: 2024-8-8-Go-Releaser.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).

toc:
  - name: Introduction
  - name: Prerequisites
  - name: Setting Up Go-Releaser in GCP Cloud Build
    subsections:
      - name: Create the Cloud Build Configuration
      - name: Configure Go-Releaser
      - name: Running the Build
  - name: Uploading Artifacts to Google Cloud Storage
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

Continuous integration and continuous delivery (CI/CD) are crucial for modern software development, and automating these processes can save significant time and reduce errors. In this guide, we'll walk through the steps to run Go-Releaser within Google Cloud Platform (GCP) Cloud Build and upload the generated artifacts into Google Cloud Storage (GCS). Each release version will be organized into its own folder in GCS, making it easy to manage and retrieve specific versions of your software.

## Prerequisites

Before we get started, ensure you have the following prerequisites in place:

- **Google Cloud Platform Account**: You need access to a GCP project with billing enabled.
- **Go-Releaser**: Installed on your local machine for initial setup and testing.
- **Google Cloud SDK**: Installed and configured on your machine.
- **Cloud Build API**: Enabled in your GCP project.
- **Google Cloud Storage Bucket**: Created for storing the release artifacts.

## Setting Up Go-Releaser in GCP Cloud Build

### 1. Create the Cloud Build Configuration

First, we'll define a `cloudbuild.yaml` file to automate the build process using GCP Cloud Build. This file describes the steps needed to run Go-Releaser and upload the artifacts.


```yaml
steps:
  # Step 1: Set up Go environment with Go 1.23.0
  - name: 'golang:1.23.0'
    id: 'setup-go'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        go version
        go mod download  # Download Go module dependencies

  # Step 2: Run GoReleaser using its Docker image
  - name: 'goreleaser/goreleaser:latest'
    id: 'run-goreleaser'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        goreleaser release --snapshot
  # Upload artifacts to Google Cloud Storage
  - id: "Upload dist to storage"
    name: 'gcr.io/google.com/cloudsdktool/cloud-sdk:slim'
    automapSubstitutions: true
    script: |
      #!/usr/bin/env bash
      if [ ! -z "$TAG_NAME" ]; then
        gcloud storage cp --recursive ./dist/* gs://$Your_Bucket_Name/main/$TAG_NAME/
      else
        gcloud storage cp --recursive ./dist/* gs://$Your_Bucket_Name/main/dist/
      fi
options:
  logging: CLOUD_LOGGING_ONLY

  pool:
    name: 'projects/$PROJECT_ID/locations/us-central1/workerPools/$Your_Private_Pool_Name'
```
- `Steps`: This block defines tasks that Cloud Build performs in sequence, from setting up the environment to uploading artifacts to GCS.
- `Name`: Specifies the Docker image used in the build process. For example, `golang:1.23.0 pulls` a Go image, while `goreleaser/goreleaser:latest` pulls the GoReleaser image.
- `Id`: Gives each step an optional identifier for logging or reference purposes.
- `Entrypoint`: Specifies the default command that will run inside the container. In this example, it is set to `bash`.
- `Args`: Contains the commands or shell script to execute. For example, `go mod download` downloads the Go dependencies, and `goreleaser release --snapshot` generates the release artifacts.
- `AutomapSubstitutions`: Allows variable substitution, e.g., for dynamic bucket or branch names.
- `Script`: Bash commands that run during the build step. For example, the `gcloud storage cp` command copies the build artifacts to a specified GCS bucket.

### 2. Create the Go Releaser Configuration
Create a `.goreleaser.yml` configuration file in your project root. This file tells Go-Releaser how to build and package your application.

```yaml
version: 2
before:
  hooks:
    - go mod tidy
builds:
  - env:
      - CGO_ENABLED=0
    goos:
      - linux
      - windows
      - darwin
    goarch:
      - amd64
      - arm
      - arm64
release:
  disable: trueES

archives:
  - format: tar.gz
    # this name template makes the OS and Arch compatible with the results of `uname`.
    name_template: >-
      {{ .ProjectName }}_
      {{- title .Os }}_
      {{- if eq .Arch "amd64" }}x86_64
      {{- else if eq .Arch "386" }}i386
      {{- else }}{{ .Arch }}{{ end }}
      {{- if .Arm }}v{{ .Arm }}{{ end }}
    # use zip for windows archives
    format_overrides:
      - goos: windows
        format: zip
changelog:
  sort: asc
  filters:
    exclude:
      - "^docs:"
      - "^test:"
```
- `Version`: The version of Go-Releaser configuration.
- `Before.hooks`: Commands to run before the main build process begins. Here, go mod tidy ensures that the Go module dependencies are clean and up-to-date.
- `Builds`: Defines the builds that Go-Releaser should produce. This includes specifying the operating systems (goos) and architectures (goarch) you want to support. In this example, Linux, Windows, and macOS are supported across different CPU architectures.
- `CGO_ENABLED=0`: This environment variable ensures that the Go build process uses pure Go, which avoids dependencies on C libraries.
- `Archives`: Specifies how the build output should be packaged (e.g., tar.gz for Unix-like systems and zip for Windows).
- `Name_template`: Defines the naming format for the output artifacts based on the OS and architecture.
- `Changelog`: Customizes how the changelog is generated. In this case, commits that start with docs: or test: are excluded from the changelog.