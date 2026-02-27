# Security Policy

## Reporting a Vulnerability

This repository contains documentation, templates, and a static HTML assessment tool. It does not include server-side code or services that process user data.

If you discover a security issue — for example, a cross-site scripting (XSS) vulnerability in the assessment tool, or guidance in the templates that would lead to an insecure configuration — please report it responsibly:

1. **Do not open a public issue.**
2. Instead, email **jknight137 [at] protonmail [dot] com** with:
   - A description of the issue
   - Steps to reproduce
   - The potential impact
3. You will receive an acknowledgment within 48 hours and a resolution timeline within 7 days.

## Scope

| In scope                                                | Out of scope                                                          |
| ------------------------------------------------------- | --------------------------------------------------------------------- |
| XSS or injection in `public/assessment/index.html`      | Vulnerabilities in third-party AI agent tools referenced in docs      |
| Template guidance that would create security weaknesses | Hypothetical risks in the maturity model's recommendations            |
| Sensitive data accidentally committed to the repo       | General questions about AI agent security (use Issues or Discussions) |

## Supported Versions

This project follows a rolling-release model. Security fixes are applied to the `main` branch only.
