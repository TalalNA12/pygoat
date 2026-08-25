# IronGate: Automated CI/CD Security Pipeline

> **IronGate** is an automated, shift-left DevSecOps pipeline implemented within GitHub Actions. It enforces defense-in-depth application security by integrating automated static analysis, dependency auditing, container scanning, secret detection, and dynamic runtime testing as mandatory quality gates before merging into the production branch.

---

## 🏛️ Architecture Overview

The pipeline intercepts vulnerabilities across the entire Software Development Life Cycle (SDLC) using a multi-stage automated workflow:

[ Developer Commit / PR ]
│
▼
┌──────────────────────────────────────────────────────────┐
│                   IronGate CI/CD Pipeline                 │
├──────────────────────────┬───────────────────────────────┤
│ 1. Secret Scanning       │ Gitleaks + Push Protection    │
│ 2. SAST                  │ Semgrep (OWASP / Django / Py) │
│ 3. Dependency (SCA)      │ pip-audit (PyPI / OSV)        │
│ 4. Container Scanning    │ Trivy (Base Image & Packages) │
│ 5. DAST                  │ OWASP ZAP (Baseline Scan)     │
└──────────────────────────┴───────────────────────────────┘
│
▼
┌──────────────────────────────────────────────────────────┐
│      Policy Gate: GitHub Branch Protection Rules         │
│  (Blocks merge to 'master' on required status failures)  │
└──────────────────────────────────────────────────────────┘


---

## 🔒 Security Stages & Controls

### 1. Secret Detection & Verification
* **Engine:** `Gitleaks` via GitHub Actions + GitHub Native Push Protection.
* **Mechanism:** Scans full commit history (`fetch-depth: 0`) and pull request diff ranges using entropy analysis and regex pattern matching.
* **Triage & Fine-Tuning:** Uses `.gitleaks.toml` with granular path and commit-hash allowlisting to prevent false positives from historical verification tests without disabling core rules.

### 2. Static Application Security Testing (SAST)
* **Engine:** `Semgrep`
* **Rulesets:** `p/owasp-top-ten`, `p/python`, `p/Django`
* **Mechanism:** Converts source code into Abstract Syntax Trees (AST) to detect structural flaws, including SQL Injection (SQLi), Command Injection, Insecure Deserialization, Cross-Site Scripting (XSS), and exposed debug configurations.

### 3. Software Composition Analysis (SCA)
* **Engine:** `pip-audit`
* **Database:** PyPI Advisory Database and Open Source Vulnerabilities (OSV).
* **Mechanism:** Scans third-party package dependencies and sub-dependencies for known CVEs and flags required patched versions.

### 4. Container Image & Configuration Scanning
* **Engine:** `Trivy`
* **Hardening Decisions:**
  * Upgraded vulnerable legacy base images to `python:3.12-slim-bookworm`.
  * Resolved Dockerfile linting finding `DS-0002` by dropping root execution and enforcing non-root runtime permissions (`USER pygoat`).
* **CI Gating:** Filtered to fail builds exclusively on `CRITICAL,HIGH` severity findings to eliminate low-signal noise.

### 5. Dynamic Application Security Testing (DAST)
* **Engine:** `OWASP ZAP` (ZAP Baseline Action)
* **Mechanism:** Spins up the application container in the background during CI, validates availability via an active `curl` health-check polling loop, and dynamically crawls the live running application for runtime issues, missing security headers, and cookie flags.

---

## 🛑 Policy Enforcement & Merge Gates

To prevent insecure code from entering production, the repository enforces **Branch Protection Rulesets** on the default branch:

* **Enforced Pull Requests:** Direct commits to `master` are blocked.
* **Required Status Checks:** Every pipeline check (`gitleaks`, `semgrep`, `pip-audit`, `trivy`, `zap`) must return a successful exit code before the merge button unlocks.
* **Fail-Closed Security:** Unpatched vulnerabilities natively halt the deployment workflow at the pull request stage.

---

## 🛠️ Tech Stack & Tooling

| Domain | Technology |
| :--- | :--- |
| **CI/CD Platform** | GitHub Actions |
| **Secret Scanning** | Gitleaks |
| **SAST** | Semgrep |
| **SCA / Dependency Audit** | pip-audit |
| **Container Scanning** | Trivy |
| **DAST** | OWASP ZAP |
| **Container Engine** | Docker |
| **Target Application** | OWASP PyGoat (Intentionally Vulnerable Lab App) |

---

> *Note: This repository uses OWASP PyGoat as a testbed application to demonstrate automated security detection, CI triage, and policy enforcement mechanisms.*