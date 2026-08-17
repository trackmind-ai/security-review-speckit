# 🛡️ security-review-speckit

**A drop-in Claude skill that turns any AI agent into a rigorous, evidence-based security reviewer.**

Point it at a repo, a release, or a pull request and get back a structured audit with exact file
paths, line numbers, vulnerable snippets, source-to-sink traces, real-world impact, and a
prioritized remediation plan — not vague "you should sanitize inputs" hand-waving.

It defaults to **read-only analysis**, so you can run it safely on any codebase without it touching
your files unless you explicitly ask for fixes.

---

## Why you'll want this

- **One skill, five review disciplines.** SAST, SCA/supply-chain, backend authorization, frontend
  XSS, and AI/LLM security — in a single reusable spec instead of five ad-hoc prompts.
- **Evidence or it didn't happen.** Every finding is forced into the same format: severity, repo,
  endpoint, file, line numbers, snippet, source→sink explanation, impact, and remediation. No
  unactionable noise.
- **It checks its own work.** A built-in validation phase re-scans for missed routes, files,
  dependencies, and sinks, retries the gaps, and ends with a confidence-rated Validation Summary
  that tells you exactly what was and wasn't covered.
- **Framework- and language-agnostic.** It discovers your routers, package managers, lockfiles,
  Dockerfiles, and CI before it reasons — Python or Node, Django or Angular, npm or Poetry.
- **Supply-chain aware.** It resolves the *authoritative* dependency source (lockfile over
  manifest) and flags dependency-confusion, lockfile drift, root containers, and CI secret leakage.
- **Built for the AI era.** First-class prompt-injection, unsafe-tool-access, vector-DB-leakage,
  and PHI/PII-exfiltration review for LLM and agentic applications.
- **Maps to OWASP.** Findings are classified by OWASP category and a clear Critical/High/Medium/Low
  severity rubric, so they slot straight into your existing risk process.

## What it reviews

| Track | What it hunts for |
|---|---|
| **Backend authorization** | Missing auth, public mutation endpoints, IDOR/BOLA, privilege escalation, role-only checks without object authorization, unsafe admin/provider access |
| **Frontend XSS & unsafe rendering** | `innerHTML`, sanitizer bypasses, iframe `src`/`srcdoc`, markdown/HTML renderers, `DOMParser`, `eval`/`new Function`, dynamic templates — traced source→sink |
| **Dependency & supply chain (SCA)** | Vulnerable/outdated/unpinned/abandoned packages, lockfile drift, dependency confusion, Docker base-image & root-user risks, CI/build-context secret leakage |
| **AI / LLM security** | Prompt injection, unsafe agent tools, raw SQL/file tools exposed to users, vector-DB data exposure, model-telemetry leakage, model output rendered as trusted HTML |
| **Validation** | Coverage re-check, automatic focused re-scans of missed areas, false-positive checks, and an overall confidence level |

## What you get back

Every finding lands in a consistent, review-ready shape:

````markdown
### [High] IDOR on invoice detail endpoint

Repository: billing-api
Endpoint/Feature: GET /api/invoices/{id}
File: billing/api/views/invoices.py
Lines: 42-58
Type: SAST
OWASP: A01:2021 - Broken Access Control

Snippet:
```code
invoice = Invoice.objects.get(pk=request.query_params["id"])
return Response(InvoiceSerializer(invoice).data)
```

Source-to-sink: user-controlled `id` → direct ORM lookup with no ownership filter → serialized back.
Why risky: any authenticated user can read any other account's invoices by changing the id.
Potential impact: cross-tenant disclosure of billing and customer data.
Recommended remediation: scope the queryset to the requesting user's account/tenant.
````

...topped with an executive summary, Critical/High findings first, and a prioritized remediation
plan.

## Repository structure

```
security-review-speckit/
├── SKILL.md     # the skill: instructions, review tracks, finding format, SpecKit templates
├── README.md    # this file
└── LICENSE      # MIT
```

## Quick start

1. **Clone it** into your skills directory:

   ```bash
   git clone https://github.com/trackmind-ai/security-review-speckit.git \
     .claude/skills/security-review-speckit
   ```

2. **Invoke it** in Claude with a request like:

   ```
   Use the security-review-speckit skill.

   Project: <name>
   Repos: <repositories>
   Branch: <branch or release>

   Run:
   1. backend authorization review
   2. frontend XSS source-to-sink review
   3. dependency and supply-chain review
   4. AI/LLM integration review, if applicable

   Return exact files, lines, snippets, impact, and remediation.
   ```

3. **Read the report**, triage Critical/High first, and re-run with "implement fixes" mode when
   you're ready for remediation.

See [SKILL.md](SKILL.md) for the full request template, per-track modules, finding format, and
severity rules.

## License

[MIT](LICENSE) © 2026 Trackmind
