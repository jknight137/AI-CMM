# AI Agent Maturity Assessment Rubric

## How to Use This Rubric

1. Assemble a cross-functional assessment team (engineering, security, platform, compliance).
2. For each of the 9 capability domains, review the scoring criteria below.
3. Gather evidence using the evidence checklist and interview questions.
4. Score each domain from 0 to 5.
5. Calculate the overall maturity score and identify gaps.
6. Use the interpretation guide to prioritize improvements.

**Scoring rule**: A domain earns a level only if **all** required practices and artifacts for that level are in place. Partial completion means the domain stays at the previous level. You cannot skip levels.

---

## Scoring Criteria by Domain

### Domain 1: Dev Workflow

| Level | Score Criteria |
|-------|---------------|
| 0 | No intentional agent usage. No approved tools list exists. |
| 1 | Some developers use agents individually. No org-level configuration or policy. |
| 2 | Approved tools list published. Devcontainers include agent tooling. CLAUDE.md or equivalent exists in repos. Org-managed API endpoints are configured. |
| 3 | Agent usage telemetry is collected and reviewed. Devcontainer images are published through Artifactory. Standardized CLAUDE.md templates exist. |
| 4 | Agent-generated code is labeled in commits/PRs. IDE agent configs are version-controlled. Cross-repo context (RAG or doc index) is available. |
| 5 | Agent configs are tuned per-team based on outcomes. New capabilities go through structured pilots. Developer feedback loops are active and acted on. |

### Domain 2: PR and Code Review

| Level | Score Criteria |
|-------|---------------|
| 0 | No review process differentiates agent-generated code. |
| 1 | Informal disclosure of agent usage in PRs. |
| 2 | PR template includes agent disclosure. Branch protection requires human approval. Agent-specific checklist exists. |
| 3 | Agent PRs are auto-labeled. Review metrics are segmented by origin. Reviewer guidance document exists. |
| 4 | Automated pre-review checks run on agent PRs. High-risk changes require additional approval. Structured agent context is included in PRs. |
| 5 | Review rigor is risk-adjusted. Expedited review path exists for low-risk agent PRs. Defect data feeds back into agent guardrails. |

### Domain 3: CI/CD

| Level | Score Criteria |
|-------|---------------|
| 0 | No agent interaction with CI/CD. |
| 1 | Developers occasionally use agents to write or debug pipelines. |
| 2 | CODEOWNERS protects workflow files. Reusable actions library exists. Pipeline authoring guide exists. |
| 3 | Agents run as isolated CI steps. Outputs are logged and auditable. Agent CI containers are from Artifactory. |
| 4 | Agent CI cost and value are measured. Agent-aware gates exist for high-risk PRs. Agents propose pipeline improvements via PRs. |
| 5 | Agents autonomously maintain CI health (with approval). Pipeline configs generated from intent. Quality parity measured. |

### Domain 4: Infrastructure as Code (IaC)

| Level | Score Criteria |
|-------|---------------|
| 0 | No IaC or no agent usage for IaC. |
| 1 | Developers use agents to generate Terraform snippets ad hoc. |
| 2 | CLAUDE.md includes IaC conventions. Linters enforced pre-merge. Wildcard IAM prohibited. Federal/commercial modules separated. |
| 3 | Agents use internal module registry. Plan output posted to PRs. Drift detection runs. Cost estimation runs. |
| 4 | OPA/Sentinel policies enforced in CI. Production IaC changes require separate approval. Federal controls enforced by policy-as-code. |
| 5 | Agents maintain IaC (upgrades, refactors) with approval. Quality metrics tracked. Agents have read-only infra state access. |

### Domain 5: Ops and Incident Response

| Level | Score Criteria |
|-------|---------------|
| 0 | No agent usage in ops. |
| 1 | Engineers occasionally use agents during incidents for ad hoc tasks. |
| 2 | Runbook safety rules exist. Agent ops access is read-only. Incident summaries are reviewed before distribution. |
| 3 | Agents assist with structured tasks (timelines, log correlation, postmortems). Agent actions are logged. Runbooks include agent steps with handoffs. |
| 4 | Agents execute pre-approved remediation actions with audit. Incident metrics track agent contribution. |
| 5 | Agents proactively identify anomalies. Remediation accuracy measured. Postmortems feed back into agent context automatically. |

### Domain 6: Security and Compliance

| Level | Score Criteria |
|-------|---------------|
| 0 | No policy on agent usage. No awareness of data classification implications. |
| 1 | Security team is aware. Informal guidance exists. |
| 2 | Acceptable use policy published. Data classification rules defined. Federal network boundaries enforced. Secrets scanning runs. API keys centrally managed. |
| 3 | API calls routed through org proxy with logging. Audit logs captured. Quarterly reviews performed. Dependency scanning integrated. |
| 4 | DLP prevents sensitive data leakage. Automated compliance scanning runs. Agent access governed by least-privilege IAM. Exception process exists. |
| 5 | Controls continuously tuned. Behavioral anomaly detection active. Org contributes to industry standards. |

### Domain 7: Knowledge Management

| Level | Score Criteria |
|-------|---------------|
| 0 | Sparse or no documentation. Agents have no useful context. |
| 1 | Developers occasionally point agents at README files. |
| 2 | CLAUDE.md exists per repo. ADRs maintained. Agents know where docs live. |
| 3 | Doc quality measured. Agents assist with doc maintenance. Searchable doc index exists. |
| 4 | RAG or knowledge retrieval system available. Curated knowledge base with editorial process. Agent docs merged into canonical base. |
| 5 | Knowledge base refined by usage patterns. Agents flag knowledge gaps. Cross-team knowledge accessible. |

### Domain 8: Governance

| Level | Score Criteria |
|-------|---------------|
| 0 | No one owns AI agent strategy. |
| 1 | Someone informally tracks agent usage. |
| 2 | Governance body exists. Approved tools list, policy, and exception process maintained. Federal governance addendum exists. |
| 3 | Regular reporting to leadership. Cost tracked and allocated. Risk register includes agent risks. Vendor assessments done. |
| 4 | Policy-as-code enforces governance. Cross-functional governance in place. Governance aligns with broader frameworks. |
| 5 | Governance processes improved by measured outcomes. Regulatory horizon scanning active. Governance is transparent org-wide. |

### Domain 9: Evaluation and Measurement

| Level | Score Criteria |
|-------|---------------|
| 0 | No data on agent usage or impact. |
| 1 | Anecdotal feedback only. |
| 2 | 5+ KPIs defined. Baselines taken. Developer surveys conducted. |
| 3 | KPIs measured monthly. Cohort comparisons done. Agent code quality measured. |
| 4 | All 12+ KPIs active. ROI analysis includes costs and benefits. Measurement drives tool decisions. |
| 5 | Measurement framework itself is evaluated. Leading indicators tracked. Methodology shared externally. |

---

## Interview Questions

Use these questions to gather evidence during the assessment. Ask the appropriate audience for each domain.

### Dev Workflow (audience: developers, tech leads)
1. What AI agent tools do you currently use in your daily workflow?
2. Are these tools on an org-approved list? Where is that list?
3. How are agent tools configured in your development environment? Is this configuration shared or personal?
4. Does your devcontainer include agent tooling out of the box?
5. Do your repos have a CLAUDE.md or equivalent instructions file? Who maintains it?
6. How do you manage the API keys or tokens for agent tools?

### PR and Code Review (audience: developers, tech leads, reviewers)
1. When you submit a PR that was generated with agent assistance, how do you indicate that?
2. Does your PR template include a field for agent disclosure?
3. Do you review agent-generated PRs differently than human-written ones? How?
4. What automated checks run on PRs before review?
5. How do you handle large diffs generated by agents?
6. Can you show me an example of an agent-generated PR and its review?

### CI/CD (audience: platform engineers, developers)
1. Are agents used in any CI/CD pipeline steps? Which ones?
2. Who can modify `.github/workflows/` files? Is this enforced?
3. Do agent CI steps run in isolated containers? What permissions do they have?
4. Do you have a library of reusable GitHub Actions? Do agents use them?
5. How do you measure the cost and time impact of agent CI steps?

### Infrastructure as Code (audience: infrastructure engineers, platform team)
1. Do agents generate or modify Terraform code? How often?
2. What guardrails exist for agent-generated IaC (linters, policy checks, plan review)?
3. How do you separate Federal and commercial infrastructure modules?
4. Do agents use your internal module registry or generate raw resources?
5. What is the approval process for agent-generated production infrastructure changes?

### Ops and Incident Response (audience: SREs, on-call engineers, incident commanders)
1. Have agents been used during incidents? In what capacity?
2. What access do agents have to production systems during incidents?
3. Do your runbooks include guidance on using agents?
4. Can agents execute any remediation actions, or are they strictly advisory?
5. How do you log agent actions during incidents?

### Security and Compliance (audience: security team, compliance team)
1. Is there a published acceptable use policy for AI agents?
2. What data classification rules apply to agent interactions?
3. How is agent API traffic routed and logged?
4. What secrets scanning and SAST tools run on agent-generated code?
5. How are Federal-specific requirements handled differently for agent usage?
6. How are AI agent API keys managed? Who has access?

### Knowledge Management (audience: developers, tech leads, documentation owners)
1. Where do agents get their context about your projects?
2. How do you maintain the accuracy of documentation that agents consume?
3. Do agents help maintain or generate documentation? How is it reviewed?
4. Do you have a searchable documentation index or RAG system?
5. How do you prevent agents from using outdated or incorrect documentation?

### Governance (audience: engineering leadership, governance body members)
1. Who is responsible for AI agent strategy and policy?
2. How are new agent tools or expanded access approved?
3. How is agent usage cost tracked and reported?
4. What is the exception process for agent usage outside of standard policy?
5. How do Federal governance requirements differ from commercial?

### Evaluation and Measurement (audience: engineering leadership, platform team)
1. What KPIs do you track for agent adoption and effectiveness?
2. How do you measure agent-generated code quality vs. human-written code?
3. How often do you review agent-related metrics?
4. Can you show me a dashboard or report on agent usage?
5. Has agent usage data ever driven a decision to change tools or practices?

---

## Evidence Checklist

For each domain, collect the following evidence before scoring.

### Dev Workflow
- [ ] Approved tools list document (URL or location)
- [ ] Devcontainer definition showing agent tooling
- [ ] Example CLAUDE.md file from an active repo
- [ ] API key management documentation or system access
- [ ] Telemetry dashboard or report (if Level 3+)

### PR and Code Review
- [ ] PR template showing agent disclosure field
- [ ] Branch protection rule configuration screenshot
- [ ] Example agent-generated PR with review comments
- [ ] Auto-labeling workflow or bot configuration (if Level 3+)
- [ ] PR metrics segmented by origin (if Level 3+)

### CI/CD
- [ ] CODEOWNERS file showing workflow protection
- [ ] Reusable actions library repo or Artifactory location
- [ ] Agent CI step definition (container image, permissions)
- [ ] CI audit log sample
- [ ] Cost/time impact data (if Level 4+)

### Infrastructure as Code
- [ ] Terraform style guide document
- [ ] Linter configuration files in repos
- [ ] CLAUDE.md with IaC-specific instructions
- [ ] Federal vs. commercial module separation evidence
- [ ] Plan comment action configuration
- [ ] OPA/Sentinel policy library (if Level 4+)

### Ops and Incident Response
- [ ] Runbook safety rules document
- [ ] IAM role definition for agent ops access
- [ ] Incident log showing agent actions (if Level 3+)
- [ ] Approved remediation action list (if Level 4+)
- [ ] Incident metrics with agent contribution data (if Level 4+)

### Security and Compliance
- [ ] Acceptable use policy document
- [ ] Data classification matrix for agent usage
- [ ] Network boundary documentation for Federal environments
- [ ] Secrets scanning configuration evidence
- [ ] API gateway/proxy logging configuration (if Level 3+)
- [ ] DLP configuration (if Level 4+)

### Knowledge Management
- [ ] Example CLAUDE.md from active repo
- [ ] ADR directory in at least one repo
- [ ] Documentation quality metrics (if Level 3+)
- [ ] RAG or knowledge retrieval system documentation (if Level 4+)

### Governance
- [ ] Governance body charter or terms of reference
- [ ] Decision log for agent-related decisions
- [ ] Federal governance addendum
- [ ] Leadership report example (if Level 3+)
- [ ] Cost allocation data (if Level 3+)

### Evaluation and Measurement
- [ ] KPI definitions document
- [ ] Baseline measurement report
- [ ] Developer survey instrument and results
- [ ] Monthly KPI report (if Level 3+)
- [ ] ROI analysis (if Level 4+)

---

## Scoring Summary Sheet

| Domain | Score (0-5) | Key Evidence | Key Gaps |
|--------|-------------|-------------|----------|
| Dev Workflow | | | |
| PR and Code Review | | | |
| CI/CD | | | |
| Infrastructure as Code | | | |
| Ops and Incident Response | | | |
| Security and Compliance | | | |
| Knowledge Management | | | |
| Governance | | | |
| Evaluation and Measurement | | | |
| **Overall Average** | | | |

---

## Interpretation Guide

### Overall Score Ranges

| Average Score | Interpretation | Recommended Action |
|--------------|---------------|-------------------|
| 0.0 - 0.9 | **No meaningful adoption.** The org either prohibits agents or is unaware of usage. | Start with awareness and a small pilot. Establish basic policy. |
| 1.0 - 1.9 | **Early exploration.** Individual usage exists but is ungoverned. Risk is unmanaged. | Prioritize governance (policy, approved tools) and security (data classification, key management). |
| 2.0 - 2.9 | **Foundation in place.** Policies and tooling exist but are not yet instrumented or measured. | Focus on instrumentation (telemetry, metrics, audit logs) and standardization (templates, reusable components). |
| 3.0 - 3.9 | **Instrumented and managed.** The org has visibility into agent usage and quality. Workflows are standardized. | Optimize: embed agents deeper in CI/CD and IaC, automate governance, invest in advanced capabilities (RAG, risk scoring). |
| 4.0 - 4.9 | **Well-optimized.** Agents are integrated across the SDLC with automated governance and measurement. | Move toward adaptive: continuous tuning, proactive capabilities, industry leadership. |
| 5.0 | **Adaptive.** Continuous improvement across all domains. | Maintain and share learnings. Watch for complacency. |

### Domain Score Gaps

- If any domain is **2+ levels below** the overall average, treat it as a critical gap requiring immediate attention.
- If Security and Compliance is below Level 2, **stop expanding agent usage** until it reaches Level 2.
- If Governance is below Level 2, policy enforcement is likely absent and other domain scores may be unreliable.

### Federal Environment Considerations

- For teams working in Federal environments, Security and Compliance and Governance must be at **Level 2 minimum** before any agent usage is permitted.
- Federal environments should be assessed separately from commercial if the tooling and policies differ.
- Document any delta between commercial and Federal scores and track them independently.
