---
layout: default
title: Installation
---

# Installation & Setup Guide

This guide walks you through how I set up the **Jenkins Pipelines Lab** environment. You can follow these steps to replicate my infrastructure locally or on a server.

## 📋 Prerequisites

Before starting, I ensure my environment meets these requirements:

*   **Linux Server:** I recommend Ubuntu 22.04 LTS or Debian 11.
*   **Jenkins:** A running instance (LTS Version).
*   **Kubernetes Cluster:** I use Minikube for local testing, but K3s or Kind work great too.
*   **Podman:** Version 4.0+ is required for the rootless builds.

---

## 🛠️ Step 1: Configure the Jenkins Agent

The Jenkins agent is the workhorse. I need to install the specific tools my pipeline relies on.

### 1. Install Podman (The Container Engine)
I use Podman instead of Docker for security.
```bash
sudo apt-get update
sudo apt-get install -y podman
# Verify installation
podman --version
```

### 2. Install Helm (The Package Manager)
I use Helm to manage Kubernetes deployments.
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 3. Install Kubectl (The Cluster CLI)
This is needed for the agent to talk to the cluster.
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

---

## 🔗 Step 2: Configure the Shared Library

Now I need to tell Jenkins where to find my pipeline logic.

1.  Navigate to **Manage Jenkins** > **System**.
2.  Scroll down to the **Global Pipeline Libraries** section.
3.  Click **Add** and configure it exactly like this:
    *   **Name:** `jenkins-pipelines-configuration` (This name is critical, it's used in the `@Library` import).
    *   **Default Version:** `master` (or the branch you want to track).
    *   **Retrieval Method:** Modern SCM.
    *   **Source Code Management:** Git.
    *   **Project Repository:** `https://github.com/aomerge-SRE-Learning/jenkins-pipelines-configuration.git`

---

## 📝 Step 3: Create a Pipeline Job

I'll create a sample job to test the setup.

1.  Create a new **Pipeline** item in Jenkins.
2.  In the **Pipeline** script section, I paste this configuration. This is all the code needed in the job itself!

```groovy
// Import my shared library
@Library('jenkins-pipelines-configuration') _

// Initialize the pipeline for an Angular application
jenkinsPipelinesExample(
    language: 'angular',
    dockerRegistry: 'docker.io/your-user',
    typeDeployd: 'helm', // Optional (defaults to helm in the library)
    configRepoUrl: 'https://github.com/<org>/config-pipelines-jenkins.git', // Recommended
    approvers: 'admin' // Optional
)
```

---

## ✅ Step 4: Run & Verify

I trigger the build manually. If everything is configured correctly, I should see the following stages in the Jenkins UI:

1.  **Checkout**
2.  **Config**
3.  **Copy values helm** (external config checkout when `configRepoUrl` is set)
4.  **Copy helm**
5.  **Test**
6.  **Build**
7.  **Approval** (only on branches that require it)
8.  **Deploy**
9.  **Remove files** (cleanup)

[Back to Home](./)
