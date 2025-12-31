---
layout: default
title: Configuration
---

# Configuration Reference

This page is a **practical reference** for configuring the lab from an application repository.

It is intentionally aligned to what currently exists in the shared library repo (`jenkins-pipelines-configuration`).

---

## ✅ Minimal Jenkinsfile

```groovy
@Library('jenkins-pipelines-configuration') _

jenkinsPipelinesExample(
  language: 'angular',
  dockerRegistry: 'docker.io/your-user',
  typeDeployd: 'helm',
  configRepoUrl: 'https://github.com/<org>/config-pipelines-jenkins.git',
  approvers: 'admin'
)
```

---

## ⚙️ Parameters supported (current behavior)

These are the parameters consumed by the shared library entrypoint (`jenkinsPipelinesExample`).

| Parameter | Required | Example | Meaning |
|---|:---:|---|---|
| `language` | ✅ | `angular` / `java` | Selects the pipeline strategy (`AngularPipeline` or `JavaPipeline`). |
| `dockerRegistry` | ✅ | `docker.io/aomerge` | Registry prefix used to tag and push images. |
| `typeDeployd` | ⛔ | `helm` or `kubectl` | Tool used in the deploy stage. For Angular the deploy path is Helm-based. |
| `configRepoUrl` | ⛔ (but recommended) | `https://.../config-pipelines-jenkins.git` | External repo containing `config/<serviceName>/deploy-helm.yaml`. |
| `approvers` | ⛔ | `admin` or `user1,user2` | Jenkins submitter(s) allowed to approve the manual `input` gate. |
| `notifyOnFailure` | ⛔ | `true` | Enables a placeholder for failure notifications (implementation is a stub today). |

Notes:
- The branch name comes from `env.BRANCH_NAME`.
- Approval uses Jenkins `input()` and is triggered by the pipeline strategy rules (e.g. `master/main`).

---

## 📦 External configuration repo (`configRepoUrl`)

When `configRepoUrl` is provided, the pipeline checks it out into a folder named `config/`.

Expected layout:

```text
config/
  <serviceName>/
    deploy-helm.yaml
```

For Angular:
- `serviceName` is read from `package.json` (`name`).
- the values file used by Helm is: `config/<serviceName>/deploy-helm.yaml`.

A minimal `deploy-helm.yaml` can contain:

```yaml
deployment:
  replicas: 2
container:
  port: 80
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
service:
  type: ClusterIP
  port: 80
  targetPort: 80
```

---

## 🔐 Jenkins credentials required

These IDs are referenced by the current shared library implementation:

- `DockerHub` (username/password): used to login and push images.
- `aomerge` (git credentials): used to checkout `configRepoUrl`.
- `k8s_token_ci` (string): Kubernetes token.
- `k8s_server_ci` (string): Kubernetes API server URL.
- `k8s_ca_data_ci` (string): Base64-encoded CA data.

---

## 🧩 Branch behavior (what the library matches)

Branch names are matched **literally** (case and separators matter):

- `dev`
- `QA`
- `master` / `main`
- `feature-*`
- `bugfix-*` (Angular strategy)
- `hotfix-*` (Angular strategy)

See the detailed behavior in the [Branching Model](./branching.html) page.

---

## Known limitations (current state)

- Strategy differences: Angular implements more explicit branch cases (`bugfix-*`, `hotfix-*`) than Java.
- Namespace mapping is strategy-specific in the current codebase.
- Notification hooks exist but are placeholders.

[Back to Home](./)
