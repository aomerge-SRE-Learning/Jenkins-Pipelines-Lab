---
layout: default
title: Features
---

# Pipeline Features

I designed this platform to be feature-rich, automating as much of the software delivery lifecycle as possible. Here is a breakdown of the key capabilities I implemented.

---

## 🔄 CI/CD Capabilities

### Automated Versioning
I hate manual versioning, so I automated it. Every build generates a unique, traceable version number:
`1.0.0-20251229.45`
*   **`1.0.0`**: Extracted directly from the project manifest (e.g., `package.json`).
*   **`20251229`**: The date of the build.
*   **`45`**: The Jenkins build number.

### Intelligent Branch Strategy
I programmed the pipeline to adapt its behavior based on the Git branch. This supports a standard GitFlow workflow:

| Branch | Environment | My Pipeline's Action |
| :--- | :--- | :--- |
| **`feature/*`** | **CI (Ephemeral)** | Builds and runs tests to validate the change fast. By default it does not deploy, keeping the cluster clean. |
| **`PRs`** | **Validation** | Runs Unit Tests and Static Analysis. Does **NOT** push artifacts or deploy, keeping the registry clean. |
| **`dev`** | **Development** | Builds, Pushes, and **Automatically Deploys** to the dev cluster. |
| **`qa`** | **QA / Staging** | Builds, Pushes, and deploys to a controlled **QA** environment (`DEPLOY_QA`) to validate integration before production. |
| **`master/main`** | **Production** | Builds, Pushes Artifacts, and requires an `APPROVAL_GATE` before `DEPLOY_PROD`. |
| **`hotfix/*`** | **Production Patch** | Runs the same test gates, promotes a fix into `master/main`, then reconciles changes back to `dev`/`qa` to prevent drift. |

---

## 🛠 Technical Innovations

### Dynamic Nginx Configuration
One challenge I faced was handling environment variables in Single Page Applications (SPAs) like Angular.
*   **My Solution:** I inject the application name and other config values into the `nginx.conf` *at build time*.
*   **Benefit:** I can use a single, generic Nginx configuration for all my frontend projects, reducing maintenance to zero.

### Self-Healing & Garbage Collection
I noticed that CI servers often crash due to full disks. To solve this, I wrote a `Trash` component that runs after every build.
*   **Workspace Cleanup:** It wipes `dist/` and `coverage/` folders.
*   **Image Pruning:** It automatically deletes old container images, keeping only the last 3 versions to save space.

### Security First Approach
Security wasn't an afterthought; it was a foundational requirement.
*   **Rootless Builds:** By using Podman, I ensure that no build process ever runs as `root`.
*   **Secret Injection:** I use Jenkins Credentials Binding to inject secrets only into the specific steps that need them, preventing leakage in logs.

[Back to Home](./)
