# Security Policy

## Reporting a vulnerability

Please do not open a public issue for a security vulnerability.

Report it privately via GitHub's [private vulnerability
reporting](https://github.com/trackmind-ai/spec-security-review/security/advisories/new), or
email **security@trackmind.com**.

Include the affected version, reproduction steps, and impact. We aim to acknowledge within
three business days and to ship a fix or mitigation for confirmed reports.

## Supported versions

The latest release on the default branch receives security fixes.

## Scope and design notes

`spec-security-review` is a single skill made of Markdown instructions. It contains no hooks,
scripts, MCP servers or installers, makes no network requests of its own, and sends nothing
to Trackmind.

**What you share with Claude.** A review works by Claude reading your repository in the
current conversation. Code and any secrets in it are seen by Claude as part of that
conversation, like any file you ask it to read. Do not point it at files you are not allowed
to share with Claude.

**Reports can contain sensitive findings.** A report quotes exact files, lines and snippets
and describes exploitable weaknesses. Store and share it as you would a vulnerability report.

**Read-only by default.** The skill edits files only if you explicitly ask it to implement
fixes.
