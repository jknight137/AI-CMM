# Reference Architecture for AI Agent Integration

## Overview

This document describes the reference architecture for integrating AI coding agents into software development workflows. It covers three deployment contexts:

1. **IDE/Devcontainer** -- agents used during local development
2. **CI/CD Pipeline** -- agents used during build, test, and deploy
3. **Ops/Incident** -- agents used during operational tasks and incident response

For each context, the document provides component diagrams (text-based), data flow descriptions, and recommended guardrails.

---

## Architecture Principles

1. **Least privilege**: Agents receive only the access they need for their specific task. No shared credentials.
2. **Audit everything**: Every agent interaction is logged with who, what, when, where, and why.
3. **Human in the loop**: Agents propose; humans approve. Exceptions require explicit pre-approval and a bounded action list.
4. **Environment separation**: Commercial and Federal environments have distinct tooling, network paths, and policies.
5. **Defense in depth**: No single control prevents misuse. Multiple layers (policy, network, IAM, scanning, review) overlap.
6. **Vendor neutrality where practical**: Core workflows should not depend on a single agent vendor's proprietary features.

---

## Context 1: IDE and Devcontainer

### Component Diagram

```
+------------------------------------------------------------------+
|  Developer Workstation / Cloud IDE                                |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |  Devcontainer (Artifactory-hosted image)                    |  |
|  |                                                              |  |
|  |  +------------------+    +-----------------------------+    |  |
|  |  |  IDE             |    |  Agent CLI (Claude Code,    |    |  |
|  |  |  (VS Code, etc.) |    |  Copilot CLI, etc.)         |    |  |
|  |  |                  |    |                             |    |  |
|  |  |  Agent Extension |    |  Reads: CLAUDE.md, code,   |    |  |
|  |  |  (Copilot, Claude|    |  ADRs, style guides        |    |  |
|  |  |   extension)     |    |                             |    |  |
|  |  +--------+---------+    +-------------+---------------+    |  |
|  |           |                            |                     |  |
|  |           +----------+    +------------+                     |  |
|  |                      |    |                                  |  |
|  |                      v    v                                  |  |
|  |              +-------+----+--------+                         |  |
|  |              |  Org API Proxy      |                         |  |
|  |              |  (routes to Bedrock |                         |  |
|  |              |   or direct API)    |                         |  |
|  |              +-------+-------------+                         |  |
|  +------------------------------------------------------------+  |
|                         |                                         |
+-----------+-------------+-----------------------------------------+
            |
            v
+-----------+-------------+     +---------------------------+
|  AWS Bedrock / Anthropic|     |  Logging and Audit        |
|  API (commercial)       |     |  (CloudWatch / SIEM)      |
+--------------------------+     +---------------------------+
            |
            |  (Federal environments use separate endpoint)
            v
+--------------------------+
|  AWS Bedrock GovCloud    |
|  (FedRAMP authorized)    |
+--------------------------+
```

### Data Flow

1. Developer works in a devcontainer built from an Artifactory-hosted image.
2. Agent tools (IDE extension, CLI) are pre-installed in the devcontainer image.
3. Agent reads project context: CLAUDE.md, source code, ADRs, style guides (all local to the repo).
4. Agent API calls route through an org-managed proxy running inside the devcontainer or on the org network.
5. The proxy:
   - Injects org-managed API credentials (developer never sees raw keys).
   - Routes commercial traffic to AWS Bedrock (or direct Anthropic API).
   - Routes Federal traffic to AWS Bedrock in GovCloud.
   - Logs all requests and responses (with PII redaction) to CloudWatch or SIEM.
6. Agent responses are rendered in the IDE or CLI. Developer accepts, modifies, or rejects.
7. Developer commits code through normal git workflow.

### Guardrails

| Guardrail | Implementation |
|-----------|---------------|
| **API credential isolation** | Credentials stored in AWS Secrets Manager. Injected at devcontainer startup via environment variable. Developers cannot extract raw keys. |
| **Network boundary (Federal)** | Federal devcontainers route traffic only to GovCloud endpoints. Commercial AI endpoints are blocked via network policy or proxy rules. |
| **Context control** | CLAUDE.md defines what context the agent should use. Agents are not given access to `.env`, secrets files, or data directories. `.gitignore`-style exclusion list for agent context. |
| **DLP (Level 4+)** | Proxy inspects outbound requests for patterns matching secrets, PII, or classified data markers. Blocks or redacts before sending. |
| **Filesystem sandboxing** | Devcontainer mounts only the project workspace. No access to host filesystem, other projects, or credential stores outside the container. |
| **Logging** | Every API call logged with: timestamp, developer identity, repo, prompt hash (not full prompt in Federal), response hash, model used, token count. |

---

## Context 2: CI/CD Pipeline

### Component Diagram

```
+------------------------------------------------------------------+
|  GitHub Actions Workflow                                          |
|                                                                   |
|  +--------------------+                                          |
|  |  Trigger:          |                                          |
|  |  pull_request      |                                          |
|  |  (on PR open/sync) |                                          |
|  +---------+----------+                                          |
|            |                                                      |
|            v                                                      |
|  +---------+----------+    +-------------------------------+     |
|  |  Standard CI Steps |    |  Agent Gate Steps             |     |
|  |  - checkout         |    |  (run in isolated container)  |     |
|  |  - build            |    |                               |     |
|  |  - unit tests       |    |  +-------------------------+  |     |
|  |  - lint             |    |  | Secrets scan (gitleaks)  |  |     |
|  |  - SAST scan        |    |  +-------------------------+  |     |
|  +--------------------+    |  | Dependency audit         |  |     |
|                             |  +-------------------------+  |     |
|                             |  | Policy check (OPA)      |  |     |
|                             |  +-------------------------+  |     |
|                             |  | Agent label check       |  |     |
|                             |  +-------------------------+  |     |
|                             +-------------------------------+     |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |  Agent CI Steps (optional, Level 3+)                        |  |
|  |  Run in isolated container from Artifactory                 |  |
|  |  No deployment credentials                                  |  |
|  |                                                              |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  |  | Test generation         |  | Documentation update    |   |  |
|  |  | (agent writes tests for |  | (agent updates README   |   |  |
|  |  |  changed code)          |  |  based on code changes) |   |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  |  | Code review assist      |  | IaC validation          |   |  |
|  |  | (agent posts review     |  | (agent checks Terraform |   |  |
|  |  |  comments on PR)        |  |  against conventions)   |   |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |  Deployment Steps (standard, no agent involvement)          |  |
|  |  - terraform plan/apply (with human approval gate)          |  |
|  |  - ECS deploy / Lambda deploy                               |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

### Data Flow

1. Developer opens a PR on GitHub.
2. GitHub Actions workflow triggers.
3. Standard CI steps run: build, test, lint, SAST.
4. Agent gate steps run in parallel:
   - Secrets scan checks for leaked credentials.
   - Dependency audit checks for vulnerable or unapproved packages.
   - Policy check (OPA) validates IaC changes against org policies.
   - Agent label check verifies that agent-generated PRs are properly labeled.
5. If the PR is from an agent or agent-assisted (detected by label or commit trailer):
   - Additional gate steps may run (stricter checks, additional approval requirements).
6. Agent CI steps (Level 3+) run in isolated containers pulled from Artifactory:
   - Test generation: agent writes additional tests for changed code, committed as suggestions.
   - Documentation update: agent proposes README updates.
   - Code review assist: agent posts review comments highlighting potential issues.
   - IaC validation: agent checks Terraform against conventions from CLAUDE.md.
7. Agent CI steps have:
   - Read access to the repo checkout.
   - Write access only to PR comments (via GitHub token scoped to `pull-requests: write`).
   - No access to deployment credentials, AWS credentials, or secrets.
8. All agent CI step outputs are logged.
9. Deployment steps (terraform apply, ECS deploy) run only after human approval and never involve agent actions.

### Guardrails

| Guardrail | Implementation |
|-----------|---------------|
| **Container isolation** | Agent CI steps run in containers with no network access except GitHub API and org API proxy. No access to other workflow steps' credentials. |
| **Credential scoping** | Agent CI steps receive a GitHub token scoped to `contents: read` and `pull-requests: write` only. No `actions: write`, no deployment tokens. |
| **Artifactory-sourced images** | All agent CI containers are pulled from Artifactory (not Docker Hub or public registries). Images are scanned by Artifactory Xray before use. |
| **Output validation** | Agent CI step outputs (suggested tests, review comments) are posted as PR comments or suggestions, not direct commits. Humans decide what to merge. |
| **Cost controls** | Agent CI steps have timeout limits. Usage is tracked per-repo. Repos can opt out of agent CI steps. |
| **High-risk path detection** | If the PR touches paths in the high-risk category list (IAM policies, auth logic, CI workflows), agent gate steps require additional approvals via CODEOWNERS. |
| **Federal pipeline separation** | Federal repos use a separate GitHub Actions runner pool (self-hosted in GovCloud). Agent CI steps in Federal pipelines route API calls to GovCloud Bedrock only. |

---

## Context 3: Ops and Incident Response

### Component Diagram

```
+------------------------------------------------------------------+
|  Incident Response Workflow                                       |
|                                                                   |
|  +--------------------+                                          |
|  |  Alert fires       |                                          |
|  |  (CloudWatch,      |                                          |
|  |   Datadog, PagerD.)|                                          |
|  +---------+----------+                                          |
|            |                                                      |
|            v                                                      |
|  +---------+---------------------------------------------+       |
|  |  Incident Commander (Human)                           |       |
|  |                                                        |       |
|  |  Decides whether to engage agent assistance            |       |
|  +---------+---------------------------------------------+       |
|            |                                                      |
|            v                                                      |
|  +---------+---------------------------------------------+       |
|  |  Agent (Read-Only Mode)                                |       |
|  |                                                        |       |
|  |  +---------------------------+  +-------------------+  |       |
|  |  | Log search and correlation|  | Timeline builder  |  |       |
|  |  | (CloudWatch Logs,         |  | (constructs event |  |       |
|  |  |  application logs)        |  |  sequence)        |  |       |
|  |  +---------------------------+  +-------------------+  |       |
|  |  +---------------------------+  +-------------------+  |       |
|  |  | Similar incident search   |  | Draft postmortem  |  |       |
|  |  | (searches past incidents  |  | (fills template   |  |       |
|  |  |  for matching patterns)   |  |  with findings)   |  |       |
|  |  +---------------------------+  +-------------------+  |       |
|  |  +---------------------------+                         |       |
|  |  | Root cause brainstorm     |                         |       |
|  |  | (suggests hypotheses      |                         |       |
|  |  |  based on evidence)       |                         |       |
|  |  +---------------------------+                         |       |
|  +--------------------------------------------------------+       |
|            |                                                      |
|            v  (Level 4+ only, with pre-approved action list)      |
|  +---------+---------------------------------------------+       |
|  |  Agent (Controlled Write Mode)                         |       |
|  |                                                        |       |
|  |  Pre-approved actions only:                            |       |
|  |  - Restart ECS service                                 |       |
|  |  - Scale ECS task count (within limits)                |       |
|  |  - Toggle feature flag (specific flags only)           |       |
|  |  - Create JIRA ticket                                  |       |
|  |                                                        |       |
|  |  Requires: IC approval + audit log entry               |       |
|  +--------------------------------------------------------+       |
+------------------------------------------------------------------+
         |                           |
         v                           v
+------------------+       +------------------+
| CloudWatch Logs  |       | Incident Audit   |
| (read-only)      |       | Trail            |
+------------------+       +------------------+
```

### Data Flow

1. Alert fires from monitoring (CloudWatch, Datadog, PagerDuty).
2. On-call engineer and incident commander convene.
3. IC decides whether to engage agent assistance (not required, always optional).
4. Agent operates in **read-only mode** by default:
   - Searches CloudWatch logs for relevant error patterns.
   - Correlates events across services to build a timeline.
   - Searches past incident records for similar patterns.
   - Suggests root cause hypotheses based on evidence.
   - Drafts postmortem sections using the org template.
5. All agent actions are logged to the incident channel and audit trail.
6. Agent outputs are reviewed by the IC before being shared or acted on.
7. At Level 4+, with IC approval, the agent can execute pre-approved remediation actions:
   - Each action requires explicit IC confirmation.
   - Each action is logged with: who approved, what was executed, when, result.
   - Only actions from the pre-approved list are available. Anything else requires manual execution.
8. Federal incidents: agents may be restricted to read-only mode regardless of maturity level, depending on compliance requirements.

### Guardrails

| Guardrail | Implementation |
|-----------|---------------|
| **Read-only by default** | Agent IAM role for ops has only `logs:GetLogEvents`, `logs:FilterLogEvents`, `cloudwatch:GetMetricData`, and read access to incident tracker. No write permissions. |
| **Pre-approved action list** | Maintained by SRE team. Each action has: name, description, parameters, limits (e.g., max scale count), and required approver role. Stored as code in the ops repo. |
| **IC approval required** | Agent cannot execute write actions without IC typing an explicit approval command. No auto-execution. |
| **Blast radius limits** | Pre-approved actions have parameter limits (e.g., scale up by at most 2x current count). Actions outside limits require human execution. |
| **Audit trail** | Every agent action (read or write) is logged to: incident channel, CloudTrail, and internal audit system. Logs include full action details and IC approval record. |
| **Federal restrictions** | For incidents in Federal environments, agent access may be limited to log search only (no remediation actions) until FedRAMP assessment explicitly covers agent-assisted remediation. |
| **Timeout** | Agent ops sessions have a maximum duration. After timeout, the agent must be re-authorized by the IC. |

---

## Model Usage Guidance

### When to Use an Agent in IDE vs. CI vs. Ops

| Use Case | Recommended Context | Why |
|----------|-------------------|-----|
| Writing new code (feature, bug fix) | IDE / Devcontainer | Developer has full context. Agent benefits from interactive feedback. Fastest iteration loop. |
| Generating unit tests for existing code | IDE (first), then CI (for coverage gaps) | IDE for writing alongside code. CI for systematically identifying untested paths. |
| Code refactoring | IDE / Devcontainer | Requires understanding intent and project conventions. Interactive review is essential. |
| Writing Terraform modules | IDE / Devcontainer | Developer must review plan output. Requires interactive iteration. |
| PR review assistance | CI (automated) | Agent posts review comments on PRs. Humans make final decisions. |
| Secrets scanning | CI (automated) | Must run on every PR. Not interactive. |
| Dependency audit | CI (automated) | Must run on every PR. Not interactive. |
| Policy-as-code validation | CI (automated) | Must run on every IaC PR. Not interactive. |
| Documentation updates | IDE (first draft), CI (freshness checks) | IDE for substantive docs. CI for catching stale docs. |
| Log search during incidents | Ops | Requires access to production logs. Not part of development workflow. |
| Incident timeline construction | Ops | Requires access to production monitoring data. |
| Root cause brainstorming | Ops | Requires incident context and production data. |
| Production remediation | Ops (Level 4+ only) | Must be controlled, approved, and audited. Never in IDE or CI. |
| Pipeline debugging | IDE | Developer debugs locally or reads CI logs. Agent helps interpret errors. |
| ADR drafting | IDE | Architect works with agent interactively. Result is committed via normal PR. |

### Model Selection Guidance

| Task Type | Recommended Model Tier | Rationale |
|-----------|----------------------|-----------|
| Complex architecture, multi-file refactoring, IaC generation | Large (Opus, GPT-4 class) | Requires deep reasoning and broad context. |
| Standard code generation, test writing, bug fixes | Medium (Sonnet, GPT-4o class) | Good balance of quality and cost for routine tasks. |
| Code completion, simple suggestions, formatting | Small (Haiku, GPT-4o-mini class) | Fast, cheap. Quality is sufficient for autocomplete. |
| CI steps (review assist, doc update, test gen) | Medium | Cost-sensitive (runs on every PR). Quality must be reliable. |
| Ops log search, incident assist | Large | Accuracy matters most during incidents. Cost is secondary. |

---

## Network Architecture (Commercial vs. Federal)

```
Commercial Environment                    Federal Environment
========================                  ========================

Developer workstation                     Developer workstation
  |                                         |
  v                                         v
Devcontainer (Artifactory)               Devcontainer (Artifactory - GovCloud)
  |                                         |
  v                                         v
Org API Proxy (commercial VPC)           Org API Proxy (GovCloud VPC)
  |                                         |
  v                                         v
AWS Bedrock (us-east-1, us-west-2)       AWS Bedrock (us-gov-west-1)
  or Anthropic API (direct)                (FedRAMP authorized only)
  |                                         |
  v                                         v
CloudWatch Logs (commercial)             CloudWatch Logs (GovCloud)
  |                                         |
  v                                         v
SIEM (commercial)                        SIEM (GovCloud, IL4/IL5 as needed)


Key boundaries:
- Federal traffic NEVER routes through commercial endpoints
- Commercial agents NEVER access Federal data
- Separate API keys, IAM roles, and logging for each environment
- Artifactory instance or repository may be separate for Federal
```

---

## Appendix: Integration Points with Existing Tooling

| Tool | Integration Point | Notes |
|------|------------------|-------|
| **GitHub** | PR templates, CODEOWNERS, branch protection, Actions, auto-labeling | Core of the workflow. All agent guardrails are enforced here. |
| **Artifactory** | Devcontainer images, agent CI step containers, internal module registry, dependency scanning (Xray) | Artifact source of truth. No public registry pulls for agent-related containers. |
| **AWS Bedrock** | Model inference endpoint for commercial and GovCloud | Preferred over direct API calls for centralized access control and logging. |
| **AWS Secrets Manager** | API key storage for agent tools | Keys are injected at devcontainer startup and CI step initialization. |
| **CloudWatch** | Logging for API proxy, CI steps, ops agent actions | Central log aggregation. Feeds SIEM and dashboards. |
| **Terraform** | IaC generation target. Policy-as-code validation. Plan output in PRs. | Agents generate Terraform; humans review plans; CI validates policies. |
| **ECS** | Deployment target (no Kubernetes). Agent CI containers may run as ECS tasks. | For teams running CI on self-hosted infrastructure. |
| **PagerDuty/Opsgenie** | Incident trigger. Agent ops sessions are linked to incident IDs. | Ensures agent ops actions are tied to specific incidents. |
