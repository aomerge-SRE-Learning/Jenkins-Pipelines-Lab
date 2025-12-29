---
layout: default
title: Features
---

# Pipeline Features

## 🔄 CI/CD Capabilities

### Automated Versioning
Versions are generated automatically to ensure traceability:
`1.0.0-20251229.45`
*   `1.0.0`: From `package.json`
*   `20251229`: Date
*   `45`: Build Number

### Branch Strategy Support
The pipeline adapts to your Git flow:

| Branch | Environment | Action |
| :--- | :--- | :--- |
| `main` | Production | Build, Push, Deploy (w/ Approval) |
| `dev` | Development | Build, Push, Deploy (Auto) |
| `PRs` | Test | Test, Build (No Push) |

## 🛠 Technical Features

### Dynamic Nginx Configuration
For Single Page Applications (SPAs), we inject the application name into the Nginx config at build time. This allows us to use a single generic `nginx.conf` for all Angular projects.

### Self-Healing & Cleanup
The `Trash` component ensures the build server stays healthy by:
*   Cleaning up workspace artifacts (`dist`, `coverage`).
*   Pruning old container images (keeping only the last 3).

### Security First
*   **No Root Access:** All builds run as a standard user.
*   **Secret Injection:** Secrets are injected as environment variables only where needed.

[Back to Home](./)
