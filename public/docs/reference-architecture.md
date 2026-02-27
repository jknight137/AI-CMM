# Reference Architecture for AI Agent Integration

## Overview

This document describes the reference architecture for integrating AI coding agents into software development workflows. It covers three deployment contexts:

1. **IDE/Dev Environment** -- agents used during local development
2. **CI/CD Pipeline** -- agents used during build, test, and deploy
3. **Ops/Incident** -- agents used during operational tasks and incident response

For each context, the document provides component diagrams (text-based), data flow descriptions, and recommended guardrails.

---

## Architecture Principles

1. **Least privilege**: Agents receive only the access they need for their specific task. No shared credentials.
2. **Audit everything**: Every agent interaction is logged with who, what, when, where, and why.
3. **Human in the loop**: Agents propose; humans approve. Exceptions require explicit pre-approval and a bounded action list.
4. **Environment separation**: Regulated and standard environments have distinct tooling, network paths, and policies.
5. **Defense in depth**: No single control prevents misuse. Multiple layers (policy, network, IAM/RBAC, scanning, review) overlap.
6. **Vendor neutrality where practical**: Core workflows should not depend on a single agent vendor's proprietary features.

---

## Context 1: IDE and Dev Environment

### Component Diagram

```
+------------------------------------------------------------------+
|  Developer Workstation / Cloud IDE                                |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |  Dev Environment (container, Codespace, Gitpod, or local)  |  |
|  |                                                              |  |
|  |  +------------------+    +-----------------------------+    |  |
|  |  |  IDE             |    |  Agent CLI (Claude Code,    |    |  |
|  |  |  (VS Code,       |    |  Copilot CLI, etc.)         |    |  |
|  |  |   JetBrains,     |    |                             |    |  |
|  |  |   etc.)          |    |  Reads: agent instructions, |    |  |
|  |  |                  |    |  code, ADRs, style guides   |    |  |
|  |  |  Agent Extension |    |                             |    |  |
|  |  |  (Copilot, Claude|    |                             |    |  |
|  |  |   extension)     |    |                             |    |  |
|  |  +--------+---------+    +-------------+---------------+    |  |
|  |           |                            |                     |  |
|  |           +----------+    +------------+                     |  |
|  |                      |    |                                  |  |
|  |                      v    v                                  |  |
|  |              +-------+----+--------+                         |  |
|  |              |  Org API Proxy      |                         |  |
|  |              |  (routes to cloud   |                         |  |
|  |              |   model API or      |                         |  |
|  |              |   direct vendor)    |                         |  |
|  |              +-------+-------------+                         |  |
|  +------------------------------------------------------------+  |
|                         |                                         |
+-----------+-------------+-----------------------------------------+
            |
            v
+-----------+------------------+     +---------------------------+
|  Model Inference Endpoint    |     |  Logging and Audit        |
|  Examples:                   |     |  (cloud monitoring, SIEM) |
|  - AWS Bedrock               |     +---------------------------+
|  - Azure OpenAI Service      |
|  - GCP Vertex AI             |
|  - Direct vendor API         |
|  - Self-hosted (e.g., vLLM)  |
+-----------+------------------+
            |
            |  (Regulated environments use separate endpoint)
            v
+--------------------------+
|  Regulated Endpoint      |
|  Examples:               |
|  - AWS Bedrock GovCloud  |
|  - Azure Gov             |
|  - Self-hosted in        |
|    approved boundary     |
+--------------------------+
```

### Data Flow

1. Developer works in a dev environment (container, cloud IDE, or local setup) built from a registry-hosted image or configuration.
2. Agent tools (IDE extension, CLI) are pre-installed or auto-configured in the environment.
3. Agent reads project context: agent instruction files, source code, ADRs, style guides (all local to the repo).
4. Agent API calls route through an org-managed proxy running inside the environment or on the org network.
5. The proxy:
   - Injects org-managed API credentials (developer never sees raw keys).
   - Routes standard traffic to the configured model endpoint.
   - Routes regulated-environment traffic to the approved endpoint (e.g., GovCloud, gov-specific region, or self-hosted).
   - Logs all requests and responses (with PII redaction) to your monitoring/SIEM system.
6. Agent responses are rendered in the IDE or CLI. Developer accepts, modifies, or rejects.
7. Developer commits code through normal git workflow.

### Guardrails

| Guardrail | Implementation |
|-----------|---------------|
| **API credential isolation** | Credentials stored in a secrets management service (e.g., HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager). Injected at environment startup via environment variable. Developers cannot extract raw keys. |
| **Network boundary (regulated)** | Regulated-environment dev setups route traffic only to approved endpoints. Standard AI endpoints are blocked via network policy or proxy rules. |
| **Context control** | Agent instruction files define what context the agent should use. Agents are not given access to `.env`, secrets files, or data directories. `.gitignore`-style exclusion list for agent context. |
| **DLP (Level 4+)** | Proxy inspects outbound requests for patterns matching secrets, PII, or classified data markers. Blocks or redacts before sending. |
| **Filesystem sandboxing** | Containerized dev environments mount only the project workspace. No access to host filesystem, other projects, or credential stores outside the container. |
| **Logging** | Every API call logged with: timestamp, developer identity, repo, prompt hash (not full prompt in regulated environments), response hash, model used, token count. |

---

## Context 2: CI/CD Pipeline

### Component Diagram

```
+------------------------------------------------------------------+
|  CI/CD Pipeline (GitHub Actions, GitLab CI, Bitbucket Pipelines,  |
|                   Azure Pipelines, Jenkins, etc.)                 |
|                                                                   |
|  +--------------------+                                          |
|  |  Trigger:          |                                          |
|  |  Pull/Merge Request|                                          |
|  |  (on open/sync)    |                                          |
|  +---------+----------+                                          |
|            |                                                      |
|            v                                                      |
|  +---------+----------+    +-------------------------------+     |
|  |  Standard CI Steps |    |  Agent Gate Steps             |     |
|  |  - checkout         |    |  (run in isolated container)  |     |
|  |  - build            |    |                               |     |
|  |  - unit tests       |    |  +-------------------------+  |     |
|  |  - lint             |    |  | Secrets scan (gitleaks,  |  |     |
|  |  - SAST scan        |    |  |  truffleHog, or native)  |  |     |
|  +--------------------+    |  +-------------------------+  |     |
|                             |  | Dependency audit         |  |     |
|                             |  +-------------------------+  |     |
|                             |  | Policy check (OPA or     |  |     |
|                             |  |  equivalent)             |  |     |
|                             |  +-------------------------+  |     |
|                             |  | Agent label check       |  |     |
|                             |  +-------------------------+  |     |
|                             +-------------------------------+     |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |  Agent CI Steps (optional, Level 3+)                        |  |
|  |  Run in isolated container from artifact registry           |  |
|  |  No deployment credentials                                  |  |
|  |                                                              |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  |  | Test generation         |  | Documentation update    |   |  |
|  |  | (agent writes tests for |  | (agent updates README   |   |  |
|  |  |  changed code)          |  |  based on code changes) |   |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  |  | Code review assist      |  | IaC validation          |   |  |
|  |  | (agent posts review     |  | (agent checks IaC       |   |  |
|  |  |  comments on PR/MR)     |  |  against conventions)   |   |  |
|  |  +-------------------------+  +-------------------------+   |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |  Deployment Steps (standard, no agent involvement)          |  |
|  |  - IaC plan/apply (with human approval gate)                |  |
|  |  - Container deploy / serverless deploy / VM deploy         |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

### Data Flow

1. Developer opens a PR or merge request on the SCM platform.
2. CI/CD pipeline triggers.
3. Standard CI steps run: build, test, lint, SAST.
4. Agent gate steps run in parallel:
   - Secrets scan checks for leaked credentials.
   - Dependency audit checks for vulnerable or unapproved packages.
   - Policy check validates IaC changes against org policies.
   - Agent label check verifies that agent-generated PRs are properly labeled.
5. If the PR is from an agent or agent-assisted (detected by label or commit trailer):
   - Additional gate steps may run (stricter checks, additional approval requirements).
6. Agent CI steps (Level 3+) run in isolated containers pulled from your artifact registry:
   - Test generation: agent writes additional tests for changed code, committed as suggestions.
   - Documentation update: agent proposes README updates.
   - Code review assist: agent posts review comments highlighting potential issues.
   - IaC validation: agent checks IaC against conventions from agent instruction files.
7. Agent CI steps have:
   - Read access to the repo checkout.
   - Write access only to PR/MR comments (via scoped token).
   - No access to deployment credentials, cloud credentials, or secrets.
8. All agent CI step outputs are logged.
9. Deployment steps run only after human approval and never involve agent actions.

### Guardrails

| Guardrail | Implementation |
|-----------|---------------|
| **Container isolation** | Agent CI steps run in containers with no network access except SCM API and org API proxy. No access to other pipeline steps' credentials. |
| **Credential scoping** | Agent CI steps receive a token scoped to repo read and PR/MR comment write only. No deployment tokens. |
| **Registry-sourced images** | All agent CI containers are pulled from your artifact registry (not public registries without verification). Images are scanned for vulnerabilities before use. |
| **Output validation** | Agent CI step outputs (suggested tests, review comments) are posted as PR/MR comments or suggestions, not direct commits. Humans decide what to merge. |
| **Cost controls** | Agent CI steps have timeout limits. Usage is tracked per-repo. Repos can opt out of agent CI steps. |
| **High-risk path detection** | If the PR touches paths in the high-risk category list (IAM/RBAC policies, auth logic, CI workflows), agent gate steps require additional approvals via code ownership rules. |
| **Regulated pipeline separation** | Regulated-environment repos use a separate runner pool (self-hosted in the approved boundary). Agent CI steps in regulated pipelines route API calls to approved endpoints only. |

---

## Context 3: Ops and Incident Response

### Component Diagram

```
+------------------------------------------------------------------+
|  Incident Response Workflow                                       |
|                                                                   |
|  +--------------------+                                          |
|  |  Alert fires       |                                          |
|  |  (cloud monitoring,|                                          |
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
|  |  | (cloud logs, application  |  | (constructs event |  |       |
|  |  |  logs, APM traces)        |  |  sequence)        |  |       |
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
|  |  - Restart a service                                   |       |
|  |  - Scale service capacity (within limits)              |       |
|  |  - Toggle feature flag (specific flags only)           |       |
|  |  - Create incident ticket                              |       |
|  |                                                        |       |
|  |  Requires: IC approval + audit log entry               |       |
|  +--------------------------------------------------------+       |
+------------------------------------------------------------------+
         |                           |
         v                           v
+------------------+       +------------------+
| Cloud Logs       |       | Incident Audit   |
| (read-only)      |       | Trail            |
+------------------+       +------------------+
```

### Data Flow

1. Alert fires from monitoring (cloud-native monitoring, Datadog, Grafana, PagerDuty, Opsgenie, etc.).
2. On-call engineer and incident commander convene.
3. IC decides whether to engage agent assistance (not required, always optional).
4. Agent operates in **read-only mode** by default:
   - Searches logs for relevant error patterns.
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
8. Regulated-environment incidents: agents may be restricted to read-only mode regardless of maturity level, depending on compliance requirements.

### Guardrails

| Guardrail | Implementation |
|-----------|---------------|
| **Read-only by default** | Agent access role for ops has only log read, metric read, and incident tracker read permissions. No write permissions. |
| **Pre-approved action list** | Maintained by SRE team. Each action has: name, description, parameters, limits (e.g., max scale count), and required approver role. Stored as code in the ops repo. |
| **IC approval required** | Agent cannot execute write actions without IC typing an explicit approval command. No auto-execution. |
| **Blast radius limits** | Pre-approved actions have parameter limits (e.g., scale up by at most 2x current count). Actions outside limits require human execution. |
| **Audit trail** | Every agent action (read or write) is logged to: incident channel, cloud audit log, and internal audit system. Logs include full action details and IC approval record. |
| **Regulated environment restrictions** | For incidents in regulated environments, agent access may be limited to log search only (no remediation actions) until a compliance assessment explicitly covers agent-assisted remediation. |
| **Timeout** | Agent ops sessions have a maximum duration. After timeout, the agent must be re-authorized by the IC. |

---

## Model Usage Guidance

### When to Use an Agent in IDE vs. CI vs. Ops

| Use Case | Recommended Context | Why |
|----------|-------------------|-----|
| Writing new code (feature, bug fix) | IDE / Dev Environment | Developer has full context. Agent benefits from interactive feedback. Fastest iteration loop. |
| Generating unit tests for existing code | IDE (first), then CI (for coverage gaps) | IDE for writing alongside code. CI for systematically identifying untested paths. |
| Code refactoring | IDE / Dev Environment | Requires understanding intent and project conventions. Interactive review is essential. |
| Writing IaC modules | IDE / Dev Environment | Developer must review plan output. Requires interactive iteration. |
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
| Complex architecture, multi-file refactoring, IaC generation | Large (e.g., Claude Opus, GPT-4 class) | Requires deep reasoning and broad context. |
| Standard code generation, test writing, bug fixes | Medium (e.g., Claude Sonnet, GPT-4o class) | Good balance of quality and cost for routine tasks. |
| Code completion, simple suggestions, formatting | Small (e.g., Claude Haiku, GPT-4o-mini class) | Fast, cheap. Quality is sufficient for autocomplete. |
| CI steps (review assist, doc update, test gen) | Medium | Cost-sensitive (runs on every PR). Quality must be reliable. |
| Ops log search, incident assist | Large | Accuracy matters most during incidents. Cost is secondary. |

---

## Network Architecture (Standard vs. Regulated)

```
Standard Environment                     Regulated Environment
========================                 ========================

Developer workstation                    Developer workstation
  |                                        |
  v                                        v
Dev environment (registry image)         Dev environment (approved registry)
  |                                        |
  v                                        v
Org API Proxy (standard network)         Org API Proxy (regulated network)
  |                                        |
  v                                        v
Model Endpoint (standard)                Model Endpoint (approved region/boundary)
  e.g., AWS Bedrock us-east-1,             e.g., AWS Bedrock GovCloud,
  Azure OpenAI westus2,                    Azure Gov, self-hosted in
  GCP Vertex AI us-central1,               approved boundary
  or direct vendor API
  |                                        |
  v                                        v
Monitoring/Logs (standard)               Monitoring/Logs (regulated boundary)
  |                                        |
  v                                        v
SIEM (standard)                          SIEM (regulated, appropriate
                                           impact level as needed)


Key boundaries:
- Regulated traffic NEVER routes through standard endpoints
- Standard agents NEVER access regulated data
- Separate API keys, access roles, and logging for each environment
- Artifact registry instance or repository may be separate for regulated environments
```

---

## Appendix: Integration Points with Common Tooling

| Tool Category | Examples | Integration Point | Notes |
|---------------|----------|------------------|-------|
| **Source control** | GitHub, GitLab, Bitbucket, Azure DevOps | PR/MR templates, code ownership rules, branch protection, CI triggers, auto-labeling | Core of the workflow. All agent guardrails are enforced here. |
| **Artifact registry** | Artifactory, Nexus, GitHub Packages, GitLab Container Registry, AWS CodeArtifact, Azure Artifacts | Dev environment images, agent CI step containers, internal module registry, dependency scanning | Artifact source of truth. Prefer verified registry images over public registry pulls. |
| **Model inference** | AWS Bedrock, Azure OpenAI, GCP Vertex AI, direct vendor APIs, self-hosted (vLLM, Ollama) | Model inference endpoint for all agent interactions | Prefer centralized access via proxy for access control and logging. |
| **Secrets management** | HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager | API key storage for agent tools | Keys are injected at dev environment startup and CI step initialization. |
| **Monitoring** | CloudWatch, Azure Monitor, GCP Cloud Logging, Datadog, Grafana, Splunk | Logging for API proxy, CI steps, ops agent actions | Central log aggregation. Feeds SIEM and dashboards. |
| **IaC** | Terraform, OpenTofu, Pulumi, CloudFormation, Bicep | IaC generation target. Policy-as-code validation. Plan output in PRs. | Agents generate IaC; humans review plans; CI validates policies. |
| **Compute** | Kubernetes, ECS, Azure Container Apps, Cloud Run, Lambda, VMs | Deployment target. Agent CI containers may run on shared compute. | For teams running CI on self-hosted infrastructure. |
| **Incident management** | PagerDuty, Opsgenie, Incident.io, Rootly, ServiceNow | Incident trigger. Agent ops sessions are linked to incident IDs. | Ensures agent ops actions are tied to specific incidents. |
