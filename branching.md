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

*   **Fast feedback** for developers (feature branches and PR validation).
*   **Safe integration** (a dedicated QA step that behaves like production).
*   **Controlled production releases** (manual approval gate on `main`).
*   **Repeatability** (same pipeline code, different behavior based on branch context).

---

## 🧭 Branches, Environments, and What the Pipeline Does

### `feature/*`
**Purpose:** local development and isolated iteration.

**Pipeline behavior:**
*   Runs CI checks (lint/unit tests/build).
*   Produces artifacts locally in the workspace.
*   By default **does not push** to registry and **does not deploy**, to avoid polluting shared environments.

### Pull Requests (PRs)
**Purpose:** enforce quality before merging.

**Pipeline behavior:**
*   Runs the same validation gates as feature branches.
*   **Never deploys** and **never publishes images**.
*   Designed to be deterministic and safe to run often.

### `dev`
**Purpose:** continuous integration environment shared by the team.

**Pipeline behavior:**
*   Builds and tests.
*   Publishes the container image.
*   Automatically deploys to the **Development** namespace/cluster.

### `qa`
**Purpose:** pre-production validation and integration testing.

**Pipeline behavior:**
*   Builds and tests (same logic as `dev`).
*   Publishes the container image.
*   Deploys to a dedicated **QA / Staging** environment.

Why I include `qa`:
*   It gives me a stable place to run integration checks.
*   It catches configuration issues (Helm values, secrets, ingress, etc.) before production.
*   It acts as the “final rehearsal” for what `main` will do.

### `master/main`
**Purpose:** production releases (the diagram treats this as `master/main`).

**Pipeline behavior:**
*   Builds and tests (always).
*   Publishes the production-ready image.
*   Requires an **approval gate** (`APPROVAL_GATE`) before deployment.

I keep the manual gate here because production changes need a human confirmation step (even in a highly automated system).

### `hotfix/*`
**Purpose:** urgent production fixes.

**Pipeline behavior:**
*   Runs the same safety checks (`RUN_TESTS` → `PASS_TESTS`).
*   Promotes the fix into `master/main` (the diagram shows this as `BRANCH_MASTER_WITH_FIX`).
*   After production, the fix is reconciled back into the long-lived branches to avoid environment drift.

---

## 🧾 Reviews and Gates (As Shown in the Diagram)

This lab models real-world promotion control by adding explicit review steps:

*   **`REVIEW_LT`**: Team lead review / ownership check.
*   **`REVIEW_TESTER`**: QA validation step.
*   **`REVIEW_DEVOPS`**: Operational readiness (deployment/observability/config).

And a hard stop before production:

*   **`APPROVAL_GATE`**: approval required before `DEPLOY_PROD`.

---

## ✅ Naming Conventions (What I Enforce)

*   Feature branches: `feature/<ticket>-<short-description>`
*   Fix branches (optional): `hotfix/<ticket>-<short-description>`
*   Environment branches: `dev`, `qa`, `master/main`

This keeps the pipeline rules simple and avoids fragile regex logic.

---

## 🔁 Promotion Flow I Use

1.  Work happens in `feature/*`.
2.  A PR is opened and validated (tests/quality).
3.  Merge to `dev` → auto deploy to Development.
4.  Promote to `qa` → deploy to QA/Staging for integration validation.
5.  Promote to `master/main` → `APPROVAL_GATE` → deploy to Production.

---

## Notes

If you want to change the branching rules, I recommend keeping the principle: **branch name defines the operational risk**, and the pipeline should respond accordingly.

[Back to Home](./)

