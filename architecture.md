---
layout: default
title: Architecture
---

# System Architecture

When I set out to design the **Jenkins Pipelines Lab**, my primary goal was to create a system that was not just functional, but **maintainable, scalable, and secure**. I wanted to move away from the "script hell" often found in CI/CD setups and treat the pipeline infrastructure as a first-class software project.

This document details the architectural decisions I made, the patterns I implemented, and the workflows I designed.

---

## 🏗️ High-Level Design Philosophy

I adopted a **Shared Library** pattern to centralize logic. Instead of having complex `Jenkinsfiles` scattered across every microservice repository, I kept the repository configuration minimal. The heavy lifting is done by a centralized, version-controlled library that I maintain.

### The "Pipeline as Code" Concept
My approach was to treat the pipeline logic as an Object-Oriented application.
*   **Abstraction:** Developers don't need to know *how* a Docker image is built, only that it *will* be built.
*   **Inheritance:** I created base classes for common tasks and extended them for specific languages (Angular, Java).
*   **Polymorphism:** The pipeline behaves differently depending on the context (branch, language, target environment).

![High Level Architecture](./draws/Arquitectura_CI-CD.svg)

---

## 🔄 Detailed Pipeline Workflow

I designed the pipeline execution flow to be linear but flexible. Every execution goes through a standard set of stages, but the implementation of those stages varies dynamically.

![Pipeline Workflow](./draws/Arquitectura_pipelines.svg)

### 1. Initialization & Detection
The process starts when Jenkins loads the Shared Library. My code immediately:
*   **Validates the Environment:** Checks if the necessary tools (Podman, Helm, Java) are available on the agent.
*   **Detects the Language:** It looks for key files (e.g., `angular.json` for Angular, `pom.xml` for Java) to determine which pipeline strategy to instantiate.

### 2. The Build Phase (Rootless & Secure)
Security was a major focus for me. I chose **Podman** over Docker for the build process because it allows for **rootless containers**.
*   **Isolation:** The build runs inside a container that matches the build tools needed (e.g., a Node.js container for Angular apps).
*   **Artifact Generation:** The output (compiled code, binaries) is stored temporarily in the workspace.

### 3. Quality Gates
I integrated strict quality checks. The pipeline will fail immediately if:
*   Unit tests fail.
*   Code coverage drops below a defined threshold.
*   SonarQube quality gates are not met (optional integration).

### 4. Packaging & Publishing
Once the code passes all checks, I package it into an immutable container image.
*   **Tagging Strategy:** I use a combination of `version-date-buildNumber` (e.g., `1.0.0-20251230.45`) to ensure every artifact is unique and traceable.
*   **Registry:** The image is pushed to the internal registry, ready for deployment.

---

## 🚀 Deployment Strategy & Branching Model

I wanted the deployment process to be completely automated based on the Git branch, eliminating manual errors.

For a deeper explanation of the promotion rules and review gates, see the dedicated page: [Branching Model](./branching.html).

![Branching & Promotion Flow](./draws/branch..svg)

### Environment Mapping
I mapped Git branches to Kubernetes environments:

*   **`feature/*` Branches:**
    *   **Action:** CI Only (Build & Test).
    *   **Goal:** Verify code quality before merging. No deployment happens here to save resources, though ephemeral environments can be enabled if needed.

*   **`dev` Branch:**
    *   **Action:** Continuous Deployment (CD) to the **Development** namespace.
    *   **Goal:** Immediate feedback loop for developers. As soon as code is merged to `dev`, it's live in the dev environment.

*   **`qa` Branch:**
    *   **Action:** Deployment to **QA/Staging**.
    *   **Goal:** Integration validation before production (the last controlled checkpoint).

*   **`master/main` Branch:**
    *   **Action:** Staged Deployment to **Production**.
    *   **Goal:** Stability. The pipeline pauses and waits for an `APPROVAL_GATE` before deploying.

*   **`hotfix/*` Branches:**
    *   **Action:** Fast-track fixes.
    *   **Goal:** Apply urgent production patches and then reconcile changes back into `dev`/`qa` to prevent drift.

---

## 🧩 Component Interaction & Class Structure

To keep the code clean, I used Groovy's object-oriented features. This diagram shows how the different classes in my library interact.

![Component Interaction](./draws/Aruitectura_general.svg)

### Core Components

#### 1. The Orchestrator (`Main.groovy`)
This is the entry point. I designed it to be the "traffic cop". It doesn't know how to build Java or Angular; it just knows how to set up the stage and call the right specialist.

#### 2. The Strategy Pattern (`AngularPipeline` vs `JavaPipeline`)
I defined a common interface for what a pipeline should do (`build()`, `test()`, `deploy()`).
*   **`AngularPipeline.groovy`:** Implements these methods specifically for TypeScript/Node.js workflows. It knows how to run `npm install` and `ng build`.
*   **`JavaPipeline.groovy`:** Implements them for Maven/Gradle. It knows how to run `mvn clean install`.

#### 3. Configuration Management (`config/`)
I separated configuration from code.
*   **`ClusterPipeline.groovy`:** Handles Kubernetes connectivity details.
*   **`HelmPipeline.groovy`:** Manages the Helm chart values and release upgrades.

#### 4. The Cleaner (`Trash.groovy`)
I noticed that build agents often run out of disk space. I wrote this utility class to run at the end of every build (in the `post { always }` block). It aggressively cleans up:
*   Temporary workspaces.
*   Dangling container images.
*   Stopped containers.

---

## 🧠 Key Architectural Decisions

### Why Podman?
I chose Podman explicitly for its security model. In a shared CI environment, giving the Jenkins user access to the Docker daemon (which runs as root) is a massive security risk. Podman allows me to build and run containers completely in user-space.

### Why Helm?
Managing raw Kubernetes YAML files is error-prone. I created a **Generic Helm Chart** that covers 90% of our use cases.
*   **Simplicity:** Developers don't write YAML. They just pass a few parameters (image tag, port, replicas) via the pipeline.
*   **Consistency:** Every deployment looks the same, making debugging easier.

### Why Groovy Shared Libraries?
I wanted "Infrastructure as Code" to be real code, not just configuration. Groovy allows me to:
*   **Unit Test** my pipeline logic.
*   **Reuse Code** across hundreds of projects.
*   **Version Control** the pipeline definition itself.

[Back to Home](./)
