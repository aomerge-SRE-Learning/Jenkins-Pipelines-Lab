---
layout: default
title: Branching
---

# Branching Model

I designed the branching model in this lab to support a clean **promotion flow** (CI → Dev → QA → Prod) while keeping the pipeline behavior predictable and easy to operate.

The key idea is simple: **every branch maps to an environment and a specific level of automation**.

![Branching & Promotion Flow](./draws/branch..svg)

---

## 🎯 Goals of This Model

When defining the branching rules, I optimized for:

*   **Fast feedback** for developers (feature branches).
*   **Safe integration** (a dedicated QA step that behaves like production).
*   **Controlled production releases** (manual approval gate on `main`).
*   **Repeatability** (same pipeline code, different behavior based on branch context).

---

## 🧭 Branches, Environments, and What the Pipeline Does

### `feature-*`
**Purpose:** fast iteration with end-to-end validation.

**Pipeline behavior (as implemented):**
*   Runs tests and build.
*   Pushes the container image.
*   Deploys to Kubernetes.
*   Does **not** require manual approval.

### `dev`
**Purpose:** continuous integration environment shared by the team.

**Pipeline behavior:**
*   Builds and tests.
*   Publishes the container image.
*   Automatically deploys to the **Development** namespace/cluster.

### `QA`
**Purpose:** pre-production validation and integration testing.

**Pipeline behavior:**
*   Builds and tests (same logic as `dev`).
*   Publishes the container image.
*   Deploys to a dedicated **QA / Staging** environment.

Why I include `QA`:
*   It gives me a stable place to run integration checks.
*   It catches configuration issues (Helm values, secrets, ingress, etc.) before production.
*   It acts as the “final rehearsal” for what `main` will do.

### `master/main`
**Purpose:** production releases (the diagram treats this as `master/main`).

**Pipeline behavior:**
*   Builds and tests (always).
*   Publishes the production-ready image.
*   Requires a single manual approval step (Jenkins `input`) before deployment.

I keep the manual gate here because production changes need a human confirmation step (even in a highly automated system).

### `hotfix-*`
**Purpose:** urgent production fixes.

**Pipeline behavior:**
*   Runs the standard safety checks (tests/build).
*   Requires approval before deploy.
*   Implemented in the Angular strategy.

### `bugfix-*`
**Purpose:** quick validation without impacting shared infrastructure.

**Pipeline behavior (Angular strategy):**
*   Runs tests/build.
*   Does not push images.
*   Does not deploy.

---

## 🧾 Approval (What Exists Today)

The shared library currently implements **one** manual gate via Jenkins `input`:

*   It triggers for `master/main` (and for `hotfix-*` in the Angular strategy).
*   The allowed approver(s) come from `approvers` in the library config (or defaults to `admin`).

---

## ✅ Naming Conventions (What I Enforce)

*   Feature branches: `feature-<ticket>-<short-description>`
*   Fix branches: `hotfix-<ticket>-<short-description>`
*   Bugfix branches: `bugfix-<ticket>-<short-description>`
*   Environment branches: `dev`, `QA`, `master/main`

This keeps the pipeline rules simple and avoids fragile regex logic.

---

## 🔁 Promotion Flow I Use

1.  Work happens in `feature-*`.
2.  Merge to `dev` → auto deploy to Development.
3.  Promote to `QA` → deploy to QA/Staging for integration validation.
4.  Promote to `master/main` → manual approval (Jenkins `input`) → deploy to Production.

---

## Notes

If you want to change the branching rules, I recommend keeping the principle: **branch name defines the operational risk**, and the pipeline should respond accordingly.

[Back to Home](./)

