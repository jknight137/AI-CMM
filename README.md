# AI Agent Capability Maturity Model (AI-CMM)

A structured framework for organizations to adopt AI coding agents -- GitHub Copilot, Claude Code, Amazon CodeWhisperer, Cursor, and others -- safely, measurably, and at scale.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

---

## Why This Exists

AI coding agents are powerful, but uncontrolled adoption creates real risks: data leakage, ungoverned infrastructure changes, compliance gaps, and inconsistent code quality. Blanket prohibition is equally costly -- it forfeits productivity gains and pushes usage underground.

AI-CMM gives engineering leaders, platform teams, and security/compliance teams a **common vocabulary and roadmap** to move from ad-hoc experimentation to governed, optimized agent adoption.

---

## What's Included

| Resource                                                 | Description                                                                                 |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [Maturity Model](docs/ai-agent-cmm.md)                   | 6-level (0-5) model across 9 capability domains                                             |
| [Assessment Rubric](docs/assessment-rubric.md)           | Scoring criteria, evidence checklist, and interview questions for self-assessment           |
| [Interactive Assessment](public/assessment/index.html)   | Browser-based tool to score your org, generate a radar chart, and build a 90-day plan       |
| [Executive Summary](public/executive-summary/index.html) | Printable one-pager overview of the CMM for leadership and stakeholders                     |
| [Domain Deep-Dive](public/domains/index.html)            | Detailed reference for all 9 domains with practices, artifacts, and anti-patterns per level |
| [Anti-Pattern Checker](public/anti-patterns/index.html)  | Interactive diagnostic to identify common risks and get targeted mitigations                |
| [Reference Architecture](docs/reference-architecture.md) | Component diagrams and data flows for IDE, CI/CD, and Ops agent integration                 |
| [Adoption Roadmap](docs/roadmap-template.md)             | 90-day pilot + 12-month scaling plan with workstreams, KPIs, and risk register              |

### Templates (copy, customize, adopt)

| Template                                                       | Description                                                                        |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| [AI Agent Usage Policy](templates/policy-ai-agent-usage.md)    | Acceptable use policy covering approved tools, data classification, and exceptions |
| [PR Review Checklist](templates/pr-checklist.md)               | Agent-aware pull request template with disclosure and quality checks               |
| [Prompt Library](templates/prompt-library.md)                  | Curated prompts for refactoring, testing, IaC, incident triage, and more           |
| [Production Safety Runbook](templates/runbook-agent-safety.md) | 10-rule safety framework for using agents during incidents and ops tasks           |

### Implementation Examples

| Example                                                            | Description                                                                                         |
| ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| [Repo Structure](examples/repo-structure.md)                       | Recommended repo layout with agent context files (`CLAUDE.md`, `CODEOWNERS`, devcontainer config)   |
| [GitHub Actions CI Gates](examples/github-actions/agent-gates.yml) | Workflow for agent-specific PR gates: secrets scanning, dependency audit, IaC policy, risk labeling |

---

## Maturity Levels at a Glance

| Level | Name            | Summary                                                               |
| ----- | --------------- | --------------------------------------------------------------------- |
| 0     | **Unaware**     | No intentional agent use. Shadow usage may exist.                     |
| 1     | **Exploratory** | Individuals experiment. No org-level policy or tooling.               |
| 2     | **Defined**     | Policies exist. Approved tools listed. Basic guardrails in place.     |
| 3     | **Managed**     | Usage instrumented. Metrics collected. Workflows standardized.        |
| 4     | **Optimized**   | Agents integrated into CI/CD, IaC, and ops with automated governance. |
| 5     | **Adaptive**    | Continuous tuning via feedback loops and measured outcomes.           |

## Capability Domains

1. Dev Workflow
2. PR & Code Review
3. CI/CD
4. Infrastructure as Code
5. Ops & Incident Response
6. Security & Compliance
7. Knowledge Management
8. Governance
9. Evaluation & Measurement

---

## Getting Started

### Assess Your Current State

1. Read the [Maturity Model](docs/ai-agent-cmm.md) to understand the levels and domains.
2. Use the [Assessment Rubric](docs/assessment-rubric.md) or the [interactive tool](public/assessment/index.html) to score your organization.
3. Identify your lowest-scoring domains -- those are your highest-leverage improvement areas.

### Plan Your Adoption

1. Copy the [Adoption Roadmap](docs/roadmap-template.md) and fill in your org-specific details.
2. Start with Level 2 foundations: publish a [usage policy](templates/policy-ai-agent-usage.md), set up [PR checklists](templates/pr-checklist.md), and configure your [repo structure](examples/repo-structure.md).
3. Add [CI gates](examples/github-actions/agent-gates.yml) and telemetry to reach Level 3.

### Customize the Templates

All templates are in the [`templates/`](templates/) directory. They are designed to be forked, edited, and adopted as-is or adapted to your org's standards.

---

## Project Structure

```
├── docs/                   # Core framework documents
│   ├── ai-agent-cmm.md        # The maturity model
│   ├── assessment-rubric.md    # Scoring rubric
│   ├── reference-architecture.md
│   └── roadmap-template.md
├── examples/               # Implementation examples
│   ├── repo-structure.md
│   └── github-actions/
│       └── agent-gates.yml
├── public/                 # Static site (interactive tools + docs)
│   ├── anti-patterns/
│   │   └── index.html         # Anti-pattern and risk checker
│   ├── assessment/
│   │   └── index.html         # Interactive maturity assessment
│   ├── domains/
│   │   └── index.html         # Domain deep-dive reference
│   ├── executive-summary/
│   │   └── index.html         # Printable one-pager
│   ├── docs/
│   ├── examples/
│   └── templates/
├── templates/              # Ready-to-use templates
│   ├── policy-ai-agent-usage.md
│   ├── pr-checklist.md
│   ├── prompt-library.md
│   └── runbook-agent-safety.md
├── LICENSE
├── README.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
└── SECURITY.md
```

## Content Source of Truth

- Canonical content lives in `docs/`, `templates/`, and `examples/`.
- Markdown under `public/docs/`, `public/templates/`, and `public/examples/` is a mirrored copy used by the static site.
- When updating canonical content, update the corresponding `public/` mirror in the same change to avoid drift.

---

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

This work is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

You are free to share, adapt, and build upon this material for any purpose, including commercially, as long as you give appropriate credit.
