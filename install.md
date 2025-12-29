---
layout: default
title: Installation
---

# Installation & Setup Guide

This guide explains how to replicate the **Jenkins Pipelines Lab** environment.

## Prerequisites

*   **Linux Server** (Ubuntu 22.04+ recommended)
*   **Jenkins** (LTS Version)
*   **Kubernetes Cluster** (Minikube, K3s, or Kind)
*   **Podman** (v4.0+)

## Step 1: Configure Jenkins Agent

The Jenkins agent needs specific tools to execute the pipeline.

```bash
# Install Podman
sudo apt-get update
sudo apt-get install -y podman

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## Step 2: Configure Shared Library

1.  Go to **Manage Jenkins** > **System**.
2.  Scroll to **Global Pipeline Libraries**.
3.  Add a new library:
    *   **Name:** `jenkins-pipelines-configuration`
    *   **Default Version:** `master`
    *   **Retrieval Method:** Modern SCM (Git)
    *   **Project Repository:** `https://github.com/YOUR_USER/jenkins-pipelines-configuration.git`

## Step 3: Create a Pipeline Job

Create a new "Pipeline" item in Jenkins and use the following script:

```groovy
@Library('jenkins-pipelines-configuration') _

// Example for an Angular App
jenkinsPipelinesExample(
    serviceName: 'my-angular-app',
    language: 'angular',
    dockerRegistry: 'localhost',
    deployK8s: true
)
```

## Step 4: Run & Verify

Trigger the build. The pipeline should:
1.  Checkout code.
2.  Run unit tests inside a container.
3.  Build the Docker image.
4.  Deploy to your Kubernetes cluster.

[Back to Home](./)
