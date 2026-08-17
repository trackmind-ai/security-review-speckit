---
name: security-review-speckit
description: Performs reusable SAST, SCA, authorization, XSS, supply-chain, and AI/LLM security reviews. Use when the user asks for a security audit, SCA, SAST, access-control review, XSS review, dependency review, OWASP review, release security assessment, or secure remediation plan.
---

# Security Review SpecKit

## Purpose

Use this skill to perform a structured, evidence-based security review for any project or task. Default to read-only analysis unless the user explicitly asks to implement fixes.

For a ready-to-fill request template, backend/frontend/SCA modules, quick prompt, and severity rules, see the **SpecKit Templates & Modules** section at the end of this file.

## First Steps

1. Identify repositories, frameworks, package managers, route files, config files, Docker/CI files, and auth modules.
2. Determine the requested mode:
   - audit only
   - remediation plan
   - implementation
   - PR/code review
3. If scope is unclear, ask the user to choose the target:
   - backend authorization
   - frontend XSS
   - SCA/supply chain
   - AI/LLM security
   - full application review

## Review Tracks

### Backend Authorization

Enumerate routes from URL/router/controller files. For each endpoint, inspect:

- authentication classes or middleware
- permission classes
- role decorators
- object lookup fields
- user-controlled IDs
- queryset filtering
- service-layer authorization
- ownership, tenancy, assignment, or scope checks

Flag:

- missing authentication
- public mutation endpoints
- missing role checks
- role-only checks without object authorization
- IDOR/BOLA
- privilege escalation
- unsafe admin/provider/research analyst access
- operational APIs exposed to non-admin users

### Frontend XSS And Unsafe Rendering

Trace user, API, AI, markdown, rich-text, document, or config data into:

- `[innerHTML]`
- `bypassSecurityTrustHtml`
- `bypassSecurityTrustResourceUrl`
- iframe `src` or `srcdoc`
- markdown or HTML renderers
- `DOMParser`
- `ElementRef` or `nativeElement`
- direct `document` or `window` DOM writes
- `new Function`, `eval`, or dynamic templates

For each path, document:

- source
- transformations
- sanitization or missing sanitization
- sink
- exploitability
- impact
- remediation

### Dependency And Supply Chain
### Dependency Source Resolution Rules

Before performing SCA analysis, identify the actual package-management system actively used by the repository.

Determine dependency authority in this priority order:

Python:

* Pipenv → Pipfile + Pipfile.lock
* Poetry → pyproject.toml + poetry.lock
* uv → pyproject.toml + uv.lock
* pip-tools → requirements.in + compiled requirements.txt
* plain pip → requirements.txt

Node.js:

* pnpm → pnpm-lock.yaml
* yarn → yarn.lock
* npm → package-lock.json

Rules:

* Prefer lockfiles over manifest-only files whenever available.
* Do not analyze fallback manifests if a higher-priority dependency source exists.
* Do not mix dependency ecosystems unless the repository genuinely uses multiple package managers.
* Detect and report conflicting dependency sources.
* If multiple manifests exist, determine which is actively used from:

  * CI/CD pipelines
  * Dockerfiles
  * install commands
  * README/setup instructions
  * Makefiles/scripts
  * runtime configuration

Validation requirements:

* Confirm the dependency source actually used in builds and deployments.
* Explicitly state which dependency source was chosen.
* Explicitly state which files were ignored and why.
* If dependency authority cannot be determined confidently, mark findings as partial confidence.

Review:

- package manifests
- lockfiles
- Dockerfiles
- CI/CD YAML
- install commands
- package indexes
- dependency cache behavior
- base images
- build context
- secret handling

Flag:

- vulnerable packages
- outdated packages
- unpinned or wildcard dependencies
- packages installed outside lockfiles
- lockfile drift
- abandoned packages
- dependency confusion risks
- Docker images copying secrets
- containers running as root
- CI jobs exporting broad secrets

### AI And LLM Security

Inspect:

- model providers
- prompts and prompt templates
- agent tools
- SQL/database tools
- file and web access tools
- vector databases
- telemetry
- model output rendering

Flag:

- prompt injection
- unsafe tool access
- PHI/PII exfiltration
- raw SQL or file tools exposed to users
- vector DB data leakage
- LLM telemetry containing sensitive data
- model output rendered as trusted HTML

## Finding Format

Use this format for every finding:

```markdown
### [Severity] Finding Title

Repository:
Endpoint/Feature:
File:
Lines:
Type: SAST / SCA / Supply-chain / AI Security
OWASP:

Snippet:
```code
...
```

Source-to-sink or access-control explanation:
Why risky:
Potential impact:
Recommended remediation:
```

## Reporting Rules

- Put Critical and High findings first.
- Include exact file paths and line numbers.
- Include snippets, but redact secrets.
- Distinguish confirmed findings from potential/context-dependent risks.
- Do not claim the review proves no other issues exist.
- Include scan limitations when tools could not run.
- For implementation requests, produce a short remediation plan before editing.

Also add a strict validation and verification phase before finalizing the report.

Validation requirements:

* Recheck every completed finding, endpoint review, dependency review, and AI/LLM review.
* Verify whether each required check from the scope was actually covered.
* Detect missed routes, missed files, missed dependencies, missed auth flows, missed sinks, and missed privilege paths.
* If coverage is incomplete, retry the analysis automatically with focused re-scans for the missed areas.
* After retry, clearly state:

  * what was successfully validated
  * what additional findings were discovered
  * what still could not be verified
  * exact reason why verification was not possible

Add a final “Validation Summary” section containing:

1. Checks completed successfully
2. Checks retried
3. Newly identified findings after retry
4. Areas with partial confidence
5. Areas not covered
6. Reasons for non-coverage
7. False-positive checks performed
8. Overall confidence level

Also require:

* cross-validation between SAST, SCA, authorization, XSS, and AI-security findings
* verification that generated reports fully match actual code paths
* confirmation that no required OWASP category was skipped
* confirmation that all high-risk endpoints were reviewed
* explicit mention of what was done vs what was not done

If something is skipped, the report must explicitly explain:

* why it was skipped
* what evidence was unavailable
* what additional access or files are required
* estimated residual risk because of the gap

---

# SpecKit Templates & Modules

Use this SpecKit to request a reusable security audit for any repository, release, PR, or task.

## Request Template

```markdown
# Security Review Request

Project: {{PROJECT_NAME}}
Repositories:
{{REPOSITORIES}}

Branch or release: {{BRANCH_OR_RELEASE}}
Application type: {{APP_TYPE}}
Tech stack: {{TECH_STACK}}

Review mode: {{MODE}}
Options: read-only audit, remediation plan, implement fixes, PR review.

## Scope

Analyze:

- source code
- API routes, controllers, services, and middleware
- authentication and authorization flows
- shared modules and utilities
- configuration and environment files
- dependency manifests and lockfiles
- Docker, CI/CD, and infrastructure files
- third-party integrations
- AI, LLM, vector DB, or agentic workflows

## Required Checks

### SAST

- missing authentication
- missing role checks
- IDOR/BOLA
- broken access control
- privilege escalation
- injection risks
- XSS and unsafe rendering
- CSRF, CORS, and session issues
- unsafe file upload or download
- hardcoded secrets
- sensitive data exposure
- insecure logging or telemetry
- dangerous dynamic execution

### SCA And Supply Chain

- vulnerable dependencies
- outdated packages
- unpinned or wildcard dependencies
- lockfile drift
- abandoned packages
- packages installed outside lockfiles
- Docker base image risks
- build context secret leakage
- CI/CD dependency install risks
- dependency confusion risks

### AI And LLM Security

- prompt injection
- unsafe agent tools
- SQL, file, network, or browser tools exposed to users
- PHI/PII sent to external providers
- vector DB data exposure
- model telemetry leakage
- unsafe model output rendering

## Evidence Requirements

For every finding, include:

1. Finding title
2. Severity: Critical / High / Medium / Low
3. Repository
4. Endpoint or feature
5. File path
6. Line number(s)
7. Vulnerable snippet
8. Source-to-sink or access-control explanation
9. Why it is risky
10. Potential impact
11. Recommended remediation
12. SAST / SCA / Supply-chain / AI Security classification
13. OWASP category

## Output Format

Produce:

- executive summary
- critical and high findings first
- endpoint-by-endpoint backend authorization findings
- source-to-sink frontend XSS findings
- dependency and supply-chain findings
- AI/LLM security findings, if applicable
- prioritized remediation plan
- scan limitations
```

## Backend Authorization Module

Append this when you need a deep API access-control review:

```markdown
Perform a deep backend authorization and access-control review.

Enumerate all routes from router, URL, controller, and view files.

For each endpoint, identify:

- HTTP method and path
- authentication requirement
- permission class or middleware
- role checks
- object lookup fields
- user-controlled IDs
- queryset or service-layer filtering
- ownership, tenancy, assignment, or scope checks

Find:

- missing authentication
- missing role checks
- IDOR/BOLA
- privilege escalation
- unsafe admin access
- provider or staff overreach
- public mutation endpoints
- insecure operational APIs

Return endpoint-by-endpoint findings with file and line references.
```

## Frontend XSS Module

Append this when you need unsafe rendering review:

```markdown
Perform an exhaustive frontend XSS and unsafe rendering review.

Trace all user-controlled, API-controlled, AI-controlled, markdown, rich-text, document, or config data flowing into:

- innerHTML
- sanitizer bypasses
- iframe src or srcdoc
- markdown renderers
- DOMParser
- ElementRef or nativeElement
- direct document/window DOM manipulation
- new Function or eval
- dynamic templates

For every exploitable path, explain:

source -> transformation -> sanitization or missing sanitization -> sink -> impact -> fix.
```

## Backend SCA Module
### Dependency Source Resolution Rules

Before performing SCA analysis, identify the actual package-management system actively used by the repository.

Determine dependency authority in this priority order:

Python:

* Pipenv → Pipfile + Pipfile.lock
* Poetry → pyproject.toml + poetry.lock
* uv → pyproject.toml + uv.lock
* pip-tools → requirements.in + compiled requirements.txt
* plain pip → requirements.txt

Node.js:

* pnpm → pnpm-lock.yaml
* yarn → yarn.lock
* npm → package-lock.json

Rules:

* Prefer lockfiles over manifest-only files whenever available.
* Do not analyze fallback manifests if a higher-priority dependency source exists.
* Do not mix dependency ecosystems unless the repository genuinely uses multiple package managers.
* Detect and report conflicting dependency sources.
* If multiple manifests exist, determine which is actively used from:

  * CI/CD pipelines
  * Dockerfiles
  * install commands
  * README/setup instructions
  * Makefiles/scripts
  * runtime configuration

Validation requirements:

* Confirm the dependency source actually used in builds and deployments.
* Explicitly state which dependency source was chosen.
* Explicitly state which files were ignored and why.
* If dependency authority cannot be determined confidently, mark findings as partial confidence.

Append this when you need backend dependency review:

```markdown
Perform a complete backend dependency and supply-chain security review.

Analyze:

- package manifests
- lockfiles
- requirements files
- Dockerfiles
- CI/CD files
- package install commands
- public or private package indexes
- AI/LLM/vector dependencies

Identify:

- vulnerable packages
- unpinned versions
- wildcard dependencies
- abandoned packages
- dependency confusion risks
- packages installed outside lockfiles
- Docker base image and root user risks
- build context secret leakage
- CI secrets exposure
- risky AI integrations
```

## Quick Prompt

```markdown
Use the security-audit-spec skill.

Project: {{PROJECT_NAME}}
Repos: {{REPOSITORIES}}
Branch: {{BRANCH}}

Run:

1. backend authorization review
2. frontend XSS source-to-sink review
3. dependency and supply-chain review
4. AI/LLM integration review, if applicable

Return exact files, lines, snippets, impact, and remediation.
```

## Recommended Severity Rules

- Critical: unauthenticated data mutation, PHI/PII bulk exposure, account takeover, RCE, leaked live secrets, critical vulnerable dependency in reachable code.
- High: authenticated cross-tenant/object access, stored XSS, unsafe admin/provider access, webhook forgery, dangerous dynamic execution, high-impact supply-chain weakness.
- Medium: defense-in-depth gap, context-dependent XSS, outdated dependency with limited reachability, broad but authenticated metadata exposure.
- Low: hardening issue, missing security header, minor dependency hygiene issue, non-exploitable unsafe pattern.

Also add a strict validation and verification phase before finalizing the report.

Validation requirements:

* Recheck every completed finding, endpoint review, dependency review, and AI/LLM review.
* Verify whether each required check from the scope was actually covered.
* Detect missed routes, missed files, missed dependencies, missed auth flows, missed sinks, and missed privilege paths.
* If coverage is incomplete, retry the analysis automatically with focused re-scans for the missed areas.
* After retry, clearly state:

  * what was successfully validated
  * what additional findings were discovered
  * what still could not be verified
  * exact reason why verification was not possible

Add a final “Validation Summary” section containing:

1. Checks completed successfully
2. Checks retried
3. Newly identified findings after retry
4. Areas with partial confidence
5. Areas not covered
6. Reasons for non-coverage
7. False-positive checks performed
8. Overall confidence level

Also require:

* cross-validation between SAST, SCA, authorization, XSS, and AI-security findings
* verification that generated reports fully match actual code paths
* confirmation that no required OWASP category was skipped
* confirmation that all high-risk endpoints were reviewed
* explicit mention of what was done vs what was not done

If something is skipped, the report must explicitly explain:

* why it was skipped
* what evidence was unavailable
* what additional access or files are required
* estimated residual risk because of the gap
