---
layout: default
title: Home
---

# Jenkins Pipelines Lab
### Enterprise CI/CD Automation Framework

Welcome to the **Jenkins Pipelines Lab** documentation. This project demonstrates a production-ready CI/CD infrastructure designed for scalability, security, and developer experience.

---

## 🎯 Executive Summary (For Recruiters & Leads)

This lab represents a complete **DevOps Platform** implementation, not just a set of scripts. It showcases advanced capabilities in:

*   **Infrastructure as Code (IaC):** Automated pipeline logic using Groovy and Jenkins Shared Libraries.
*   **Containerization:** Secure, rootless container builds using **Podman** (replacing Docker for enhanced security).
*   **Orchestration:** Dynamic deployments to **Kubernetes** using Helm charts.
*   **Standardization:** A "Pipeline as Code" approach that allows developers to onboard new microservices (Angular, Java) with a single line of configuration.
*   **Observability & Quality:** Integrated testing, health checks, and automated artifact lifecycle management.

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

## 🚀 Project Highlights

### 1. Modular Shared Library Architecture
Instead of copying `Jenkinsfiles` across repositories, this project uses a centralized **Object-Oriented** library.
- **Benefit:** Updates to the pipeline logic (e.g., a security patch) are applied instantly to all projects.
- **Code:** `src/org/aomerge/` contains logic separated by concern (Build, Deploy, Config).

### 2. Dynamic Environment Management
The pipeline automatically detects the Git branch and adapts its behavior:
- **`main`**: Deploys to Production with approval gates.
- **`dev`**: Deploys to Development automatically.
- **`feature/*`**: Runs tests and builds ephemeral environments.

### 3. Advanced Security Patterns
- **Ephemeral Kubeconfigs:** Kubernetes credentials exist only in memory during deployment.
- **Rootless Containers:** Builds run without root privileges using Podman.
- **Secret Management:** Credentials are abstracted via Jenkins credentials store.

---

## 📚 Documentation

Explore the detailed technical documentation:

*   [**Architecture Overview**](./architecture.html): Deep dive into the system design and flow.
*   [**Installation Guide**](./install.html): How to set up this lab locally.
*   [**Pipeline Features**](./features.html): Detailed breakdown of pipeline capabilities.

---

## 👨‍💻 About the Author

This lab was engineered to demonstrate proficiency in modern SRE and DevOps practices. It solves real-world problems like configuration drift, "it works on my machine" syndrome, and deployment bottlenecks.

[View on GitHub](https://github.com/aomerge-SRE-Learning/jenkins-pipelines-configuration)
