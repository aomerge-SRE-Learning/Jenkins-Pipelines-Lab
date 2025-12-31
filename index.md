---
layout: default
title: Home
---

# Jenkins Pipelines Lab
### Enterprise CI/CD Automation Framework

Welcome to the documentation for my **Jenkins Pipelines Lab**. This project represents my implementation of a production-ready CI/CD infrastructure, designed from the ground up for scalability, security, and an optimal developer experience.

![Architecture Overview](./draws/Aruitectura_general.svg)

---

## 🎯 Project Overview

This lab is more than just a collection of scripts; it is a complete **DevOps Platform** that I engineered to solve real-world challenges. My goal was to demonstrate advanced capabilities in:

*   **Infrastructure as Code (IaC):** I automated the pipeline logic using Groovy and Jenkins Shared Libraries.
*   **Containerization:** I implemented secure, rootless container builds using **Podman** (replacing Docker for enhanced security).
*   **Orchestration:** I configured dynamic deployments to **Kubernetes** using Helm charts.
*   **Standardization:** I adopted a "Pipeline as Code" approach, allowing developers to onboard new microservices (Angular, Java) with a single line of configuration.
*   **Observability & Quality:** I integrated testing, health checks, and automated artifact lifecycle management.

### Key Technologies Stack
| Category | Technologies |
|----------|--------------|
| **CI/CD** | Jenkins, Groovy (Shared Libs), Git |
| **Containers** | Podman (Rootless), Docker |
| **Orchestration** | Kubernetes (K8s), Helm |
| **Web Server** | Nginx (Dynamic Configuration) |
| **Languages** | Java, TypeScript (Angular), Groovy |
| **OS** | Linux (Ubuntu/Debian) |

---

## 🚀 Key Features I Implemented

### 1. Modular Shared Library Architecture
Instead of copying `Jenkinsfiles` across repositories, I designed a centralized **Object-Oriented** library.
- **Benefit:** Updates to the pipeline logic (e.g., a security patch) are applied instantly to all projects.
- **Code:** `src/org/aomerge/` contains the logic I wrote, separated by concern (Build, Deploy, Config).

### 2. Dynamic Environment Management
I programmed the pipeline to automatically detect the Git branch and adapt its behavior:
- **`feature-*`**: Runs CI, pushes an image, and deploys (no manual approval).
- **`dev`**: Deploys automatically to Development.
- **`QA`**: Deploys to QA/Staging (as implemented in the current library).
- **`master/main`**: Deploys to Production behind a single manual approval step (Jenkins `input`).
- **`hotfix-*`**: Treated as a production hotfix flow (approval required). *(Implemented in the Angular pipeline strategy.)*
- **`bugfix-*`**: CI-only validation without pushing images and without deploying. *(Implemented in the Angular pipeline strategy.)*

Important detail: this lab’s shared library uses the branch names **exactly as written above** (for example, `QA` in uppercase and `feature-...` with a dash).

### 3. Advanced Security Patterns
- **Ephemeral Kubeconfigs:** I ensured Kubernetes credentials exist only in memory during deployment.
- **Rootless Containers:** Builds run without root privileges using Podman.
- **Secret Management:** Credentials are abstracted via Jenkins credentials store.

### 4. External Configuration (Helm Values)
To keep application repos clean, I separated deployment configuration from application code.

- The pipeline expects Helm values under `config/<serviceName>/deploy-helm.yaml`.
- In the current implementation this is typically provided by an external configuration repository via `configRepoUrl`.

---

## 📐 Architecture Diagrams

I have documented the detailed architectural decisions, workflows, and component interactions in the [Architecture Overview](./architecture.html). There, you will find deep dives into how I structured the pipeline logic and deployment strategies.

---

## 📚 Documentation

Explore the detailed technical documentation I have prepared:

*   [**Architecture Overview**](./architecture.html): Deep dive into the system design and flow.
*   [**Branching Model**](./branching.html): Branching and promotion flow across `feature-*`, `dev`, `QA`, `master/main`, and `hotfix-*`.
*   [**Configuration Reference**](./configuration.html): Shared library parameters, credentials, and external config layout.
*   [**Installation Guide**](./install.html): How to set up this lab locally.
*   [**Pipeline Features**](./features.html): Detailed breakdown of pipeline capabilities.

---

## 👨‍💻 About Me

I engineered this lab to demonstrate my proficiency in modern SRE and DevOps practices. It addresses common pain points I've observed, such as configuration drift, "it works on my machine" syndrome, and deployment bottlenecks.

[View on GitHub](https://github.com/aomerge-SRE-Learning/jenkins-pipelines-configuration)
