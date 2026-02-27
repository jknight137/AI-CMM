# AI Agent Capability Maturity Model (CMM)

## Assumptions

This model is designed to be platform-agnostic. Adapt it to your organization's technology stack. Common choices include:

- **Cloud**: AWS, Azure, GCP, or private cloud. Multi-cloud organizations should assess each environment separately where tooling differs.
- **Source control and CI/CD**: GitHub (GitHub Actions), GitLab (GitLab CI), Bitbucket (Bitbucket Pipelines), Azure DevOps, or other platforms.
- **Artifact management**: Artifactory, Nexus, GitHub Packages, GitLab Container Registry, AWS CodeArtifact, Azure Artifacts, or similar.
- **Development environments**: Development containers (devcontainers), GitHub Codespaces, GitLab Web IDE, Gitpod, JetBrains remote development, or other remote/containerized dev environments.
- **Container orchestration**: Kubernetes, Amazon ECS, Azure Container Apps, Google Cloud Run, serverless compute, or VM-based deployments.
- **Compliance**: Organizations may operate under one or more regulatory frameworks: FedRAMP, FISMA, HIPAA, SOC 2, PCI-DSS, GDPR, ISO 27001, or industry-specific standards. Regulated environments have stricter data residency, logging, and access-control requirements.
- **AI agents in scope**: IDE-integrated agents (Copilot, Claude Code, Cursor, Cody), CI-embedded agents (Codex, Claude Code in pipelines), and operational agents used for incident response or infrastructure changes.
- **Organizational posture**: The organization wants to adopt AI agents deliberately, with measurable guardrails, not by blanket prohibition or uncontrolled adoption.

---

## Maturity Levels

| Level | Name | Summary |
|-------|------|---------|
| 0 | Unaware | No intentional use of AI agents. Shadow usage may exist. |
| 1 | Exploratory | Individuals experiment with AI agents. No org-level policy or tooling. |
| 2 | Defined | Policies exist. Approved tools are listed. Basic guardrails are in place. |
| 3 | Managed | Agent usage is instrumented. Metrics are collected. Workflows are standardized. |
| 4 | Optimized | Agents are integrated into CI/CD, IaC, and ops workflows with automated governance. |
| 5 | Adaptive | The org continuously tunes agent behavior, context, and guardrails using feedback loops and measured outcomes. |

---

## Capability Domains

1. **Dev Workflow** -- How agents are used during local development (IDE, dev environments, CLI).
2. **PR and Code Review** -- How agent-generated code is reviewed, labeled, and merged.
3. **CI/CD** -- How agents participate in build, test, deploy pipelines.
4. **Infrastructure as Code (IaC)** -- How agents generate, modify, and validate Terraform/OpenTofu/Pulumi/CloudFormation.
5. **Ops and Incident Response** -- How agents assist during incidents, runbooks, and operational tasks.
6. **Security and Compliance** -- How agent usage meets security policies, data classification, and regulatory requirements.
7. **Knowledge Management** -- How agents consume and contribute to org documentation, ADRs, and context.
8. **Governance** -- How the org manages agent access, approvals, audit, and exceptions.
9. **Evaluation and Measurement** -- How the org measures agent value, quality, and risk.

---

## Domain-Level Detail

### Domain 1: Dev Workflow

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: Developers use personal API keys for commercial AI services. No awareness of data exposure risks.

#### Level 1 -- Exploratory
- **Required practices**: Some developers use IDE-integrated agents (Copilot, Claude Code) for code completion and generation.
- **Required artifacts**: None required; informal sharing of tips may exist.
- **Anti-patterns**: No distinction between approved and unapproved tools. Proprietary code is sent to unknown third-party endpoints.

#### Level 2 -- Defined
- **Required practices**:
  - Approved agent tools list is published and maintained.
  - Development environment definitions include agent tooling and configuration (extensions, CLI tools, project-level config files).
  - Developers use only org-approved API endpoints (e.g., cloud-hosted model APIs or an org-managed proxy).
  - Project-level agent instructions exist in repositories (e.g., CLAUDE.md, .github/copilot-instructions.md, or equivalent).
- **Required artifacts**:
  - Approved tools list (living document or wiki page).
  - Development environment configuration with agent tooling pre-installed.
  - Repository-level agent instructions file.
- **Anti-patterns**: Approving tools but not configuring them; developers still use personal accounts. Dev environments exist but do not include agent tooling.

#### Level 3 -- Managed
- **Required practices**:
  - Agent usage telemetry is collected (opt-in or org-managed): number of completions, acceptances, languages, repos.
  - Development environment images are built and published through your artifact registry with agent tooling baked in.
  - Standardized agent instruction templates are maintained per project type (service, library, IaC module).
  - Context management is intentional: agents are given curated docs, not raw repo dumps.
- **Required artifacts**:
  - Telemetry dashboard (e.g., cloud monitoring service, Datadog, Grafana, or internal tool).
  - Registry-hosted dev environment images with version tags.
  - Agent instruction template library.
- **Anti-patterns**: Collecting telemetry but never reviewing it. Dev environment images are stale and lag behind tooling updates.

#### Level 4 -- Optimized
- **Required practices**:
  - Agents are embedded in developer workflows end-to-end: scaffold, implement, test, document.
  - Agent-generated code is automatically labeled in commits and PRs (trailers, labels, or metadata).
  - IDE agent configurations are version-controlled and deployed via dev environment features or setup scripts.
  - Cross-repo context is available to agents through indexed documentation or retrieval-augmented generation (RAG).
- **Required artifacts**:
  - Commit trailer convention (e.g., `Co-Authored-By: Claude <noreply@anthropic.com>`).
  - Dev environment feature or configuration package for agent setup.
  - Documentation index or RAG pipeline.
- **Anti-patterns**: Agents have unrestricted filesystem or network access in dev environments. No differentiation between agent-assisted and human-written code.

#### Level 5 -- Adaptive
- **Required practices**:
  - Agent configurations are tuned per-team and per-project based on measured quality and velocity outcomes.
  - New agent capabilities (tool use, multi-file edits, autonomous tasks) are evaluated through structured pilots before broad rollout.
  - Feedback loops exist: developers rate agent suggestions, and those ratings inform prompt and context improvements.
- **Required artifacts**:
  - Agent configuration tuning log or changelog.
  - Pilot evaluation reports.
  - Developer satisfaction survey results (quarterly or per-pilot).
- **Anti-patterns**: Adopting every new agent feature on day one without evaluation. Ignoring negative developer feedback about agent quality.

---

### Domain 2: PR and Code Review

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: Agent-generated code is committed without any review. Reviewers do not know which code was agent-generated.

#### Level 1 -- Exploratory
- **Required practices**: Some developers disclose agent usage in PR descriptions informally.
- **Required artifacts**: None required.
- **Anti-patterns**: Agent-generated PRs are merged without human review. Large agent-generated diffs are rubber-stamped.

#### Level 2 -- Defined
- **Required practices**:
  - PR template includes a checkbox or field for declaring agent usage.
  - Agent-generated PRs require at least one human reviewer.
  - PR checklist includes agent-specific items (see `templates/pr-checklist.md`).
- **Required artifacts**:
  - PR template with agent disclosure field.
  - Branch protection or merge request approval rules enforcing at least one approval.
- **Anti-patterns**: Checklist exists but reviewers skip agent-specific items. No difference in review rigor for agent-generated code.

#### Level 3 -- Managed
- **Required practices**:
  - Agent-generated PRs are automatically labeled (via CI automation or bot).
  - Review metrics are tracked separately for agent-generated vs. human-written PRs: time-to-merge, defect rate, rework rate.
  - Reviewers receive guidance on what to look for in agent-generated code (over-abstraction, hallucinated imports, incorrect error handling).
- **Required artifacts**:
  - CI automation or bot that auto-labels agent PRs.
  - Review guidance document.
  - Dashboard splitting PR metrics by agent vs. human origin.
- **Anti-patterns**: Auto-labeling is noisy or inaccurate, causing label fatigue. Metrics are collected but not acted on.

#### Level 4 -- Optimized
- **Required practices**:
  - Automated pre-review checks run on agent-generated PRs: lint, type-check, test coverage delta, dependency audit, secrets scan.
  - High-risk changes (see High-Risk Change Categories below) from agents require additional reviewer or team lead approval.
  - Agent-generated PRs include structured context: the prompt used, files read, and changes made.
- **Required artifacts**:
  - CI gate configuration for agent PRs (see `examples/ci-gates/`).
  - Code ownership rules for high-risk paths (e.g., CODEOWNERS, GitLab Code Owners, or equivalent).
  - Structured PR description template showing agent context.
- **Anti-patterns**: Over-reliance on automated checks; humans stop reading diffs. Code ownership rules are outdated and do not reflect actual ownership.

#### Level 5 -- Adaptive
- **Required practices**:
  - Review rigor is dynamically adjusted based on change risk score (file sensitivity, blast radius, historical defect rate).
  - Agent-generated PRs that pass all automated checks and touch only low-risk files may follow an expedited review path.
  - Defect data feeds back into agent prompts and guardrails to reduce recurring issues.
- **Required artifacts**:
  - Risk scoring model or heuristic for PRs.
  - Expedited review policy with clear boundaries.
  - Feedback loop documentation showing prompt/guardrail changes driven by defect data.
- **Anti-patterns**: Expedited review becomes the default, bypassing human judgment. Risk scoring is never recalibrated.

---

### Domain 3: CI/CD

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: No CI/CD exists, or agents have no interaction with pipelines.

#### Level 1 -- Exploratory
- **Required practices**: Developers occasionally ask agents to help write or debug CI/CD pipelines.
- **Required artifacts**: None required.
- **Anti-patterns**: Agent-generated pipeline files are committed without testing. Agents suggest using security-sensitive triggers or configurations without understanding the risk.

#### Level 2 -- Defined
- **Required practices**:
  - Agents are not permitted to modify CI/CD pipeline definitions without human review (enforced via code ownership rules on pipeline config directories).
  - A library of approved, reusable pipeline components exists and agents are instructed to use them.
  - Pipeline definitions follow a standard structure documented in agent instructions or a contributing guide.
- **Required artifacts**:
  - Code ownership rules for pipeline configuration files.
  - Reusable pipeline component library (in a shared repo or artifact registry).
  - Pipeline authoring guide.
- **Anti-patterns**: Agents freely create new pipelines without review. No reusable components; every repo has bespoke pipeline logic.

#### Level 3 -- Managed
- **Required practices**:
  - Agents run as CI steps for specific tasks: generating test cases, running linters with auto-fix, updating documentation.
  - Agent CI steps run in isolated containers with no access to deployment credentials.
  - Agent CI step outputs are logged and auditable.
- **Required artifacts**:
  - Agent CI step definitions (containerized, least-privilege).
  - Audit log configuration (CI logs retained, forwarded to your monitoring/SIEM system).
  - Container images for agent CI steps published in your artifact registry.
- **Anti-patterns**: Agent CI steps have access to production secrets. Agent CI outputs are not logged. Agent containers are pulled from public registries without verification.

#### Level 4 -- Optimized
- **Required practices**:
  - Agents can propose pipeline improvements (caching, parallelism, dependency updates) as PRs, subject to human review.
  - Agent-in-CI usage is measured: cost per run, time added to pipeline, value delivered (bugs caught, tests generated).
  - Deployment pipelines include agent-aware gates: if a PR is agent-generated and touches high-risk paths, additional checks run.
- **Required artifacts**:
  - Cost and value dashboard for agent CI steps.
  - Agent-aware gate configuration.
  - Pipeline improvement PR history.
- **Anti-patterns**: Agent CI steps add significant time to pipelines without measurable value. Agents propose pipeline changes that break other repos using shared workflows.

#### Level 5 -- Adaptive
- **Required practices**:
  - Agents autonomously maintain CI health: update pinned dependency versions, rotate test fixtures, fix flaky tests (with human approval on the resulting PR).
  - Pipeline configurations are generated from higher-level intent files, with agents handling the translation.
  - Agent CI contributions are measured against the same quality bar as human contributions.
- **Required artifacts**:
  - Intent-based pipeline definition format (if applicable).
  - Autonomous maintenance policy with approval requirements.
  - Quality parity report (agent vs. human CI contributions).
- **Anti-patterns**: Agents make autonomous changes to production pipelines without approval. "Intent-based" pipelines become an abstraction that nobody understands.

---

### Domain 4: Infrastructure as Code (IaC)

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: Infrastructure is manually provisioned. No IaC exists.

#### Level 1 -- Exploratory
- **Required practices**: Developers use agents to generate IaC snippets or modules in their IDE.
- **Required artifacts**: None required.
- **Anti-patterns**: Agent-generated IaC is applied without `plan` review. Agents generate resources with overly permissive IAM/RBAC policies (e.g., `"Action": "*"`, `Owner` role on a subscription, etc.).

#### Level 2 -- Defined
- **Required practices**:
  - Agents are instructed (via project-level config or equivalent) to follow org IaC conventions: module structure, naming, tagging, backend configuration.
  - Agent-generated IaC must pass validation and a linter (tflint, checkov, tfsec, or equivalent) before merge.
  - Agents are prohibited from generating identity/access policies with wildcard actions or resources unless explicitly approved.
  - Regulated environment modules are separated from standard modules, with distinct variable files and backend configurations.
- **Required artifacts**:
  - IaC style guide.
  - Linter configuration files (checked into repos).
  - Agent instructions with IaC-specific rules.
  - Separate module paths or workspaces for regulated vs. standard environments.
- **Anti-patterns**: Style guide exists but agents are not given it as context. Linters run but failures are ignored. Regulated and standard IaC is co-mingled without clear boundaries.

#### Level 3 -- Managed
- **Required practices**:
  - Agents generate IaC using an org-approved module registry (hosted in your artifact registry or an internal repository).
  - `plan` output is posted to PRs automatically; agent-generated IaC PRs require plan review before approval.
  - Drift detection runs on a schedule; agents can propose remediation PRs.
  - Cost estimation (Infracost or similar) runs on agent-generated IaC PRs.
- **Required artifacts**:
  - Internal module registry.
  - CI automation for posting plan output as PR comments.
  - Drift detection pipeline.
  - Cost estimation integration.
- **Anti-patterns**: Agents generate raw resources instead of using approved modules. Plan output is posted but reviewers do not read it. Cost estimates are ignored.

#### Level 4 -- Optimized
- **Required practices**:
  - Agents can generate complete, compliant modules from high-level requirements (e.g., "create a storage bucket for log storage with regulatory compliance controls").
  - Generated modules are automatically validated against OPA/Sentinel/custom policies before plan.
  - Agent-generated IaC changes to production environments require a separate approval from the infrastructure team.
  - Regulatory controls (encryption, logging, access restrictions) are enforced by policy-as-code, not by relying on the agent to remember them.
- **Required artifacts**:
  - Policy-as-code library (OPA, Sentinel, or equivalent).
  - Approval workflow for production IaC changes.
  - Policy-as-code enforcement in CI.
- **Anti-patterns**: Policy-as-code exists but is not enforced in CI. Agents bypass the module registry for "simple" resources. Production approval is a rubber stamp.

#### Level 5 -- Adaptive
- **Required practices**:
  - Agents maintain IaC: upgrade provider versions, refactor deprecated resources, update modules to match new compliance requirements.
  - IaC generation quality is measured: plan accuracy, policy violations caught, rework rate.
  - Agent context includes live infrastructure state (read-only) to inform generation decisions.
- **Required artifacts**:
  - IaC maintenance automation with approval gates.
  - Quality metrics dashboard for agent-generated IaC.
  - Read-only infrastructure state access for agents.
- **Anti-patterns**: Agents have write access to infrastructure state. Automated upgrades are applied without testing. Quality metrics show declining quality but no action is taken.

---

### Domain 5: Ops and Incident Response

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: No runbooks exist. Incident response is ad hoc.

#### Level 1 -- Exploratory
- **Required practices**: On-call engineers occasionally use agents to search logs, draft incident summaries, or brainstorm root causes.
- **Required artifacts**: None required.
- **Anti-patterns**: Agents are given production credentials during incidents. Agent suggestions are followed without verification during high-stress incidents.

#### Level 2 -- Defined
- **Required practices**:
  - Agent usage during incidents follows the runbook safety rules (see `templates/runbook-agent-safety.md`).
  - Agents used in ops contexts have read-only access to logs and metrics (via your cloud monitoring service). No write access to production systems.
  - Incident summaries generated by agents are reviewed by the incident commander before distribution.
- **Required artifacts**:
  - Runbook safety rules for agent usage.
  - Read-only access role for agent ops access.
  - Incident summary review checklist.
- **Anti-patterns**: Agents have the same credentials as the on-call engineer. Agent-generated incident summaries are sent to stakeholders without review.

#### Level 3 -- Managed
- **Required practices**:
  - Agents assist with structured incident tasks: timeline construction, log correlation, similar-incident search, draft postmortems.
  - Agent actions during incidents are logged to the incident channel and audit trail.
  - Runbooks include agent-assisted steps with clear handoff points (agent proposes, human executes).
- **Required artifacts**:
  - Agent-assisted runbook templates.
  - Incident audit log including agent actions.
  - Postmortem template with agent-contribution section.
- **Anti-patterns**: Agents replace human judgment during incidents. Agent actions are not logged. Postmortems do not capture whether agent assistance helped or hindered.

#### Level 4 -- Optimized
- **Required practices**:
  - Agents can execute pre-approved remediation actions (restart a service, scale up capacity, toggle a feature flag) through a controlled interface with approval and audit.
  - Agent remediation actions are constrained to a pre-approved list; anything outside the list requires human execution.
  - Incident response metrics track agent contribution: time-to-detection, time-to-mitigation, accuracy of root cause suggestions.
- **Required artifacts**:
  - Approved remediation action list with execution interface.
  - Approval and audit trail for agent-executed remediations.
  - Incident metrics dashboard with agent contribution data.
- **Anti-patterns**: Agents execute unapproved actions. The approved action list is never updated. Agent remediation is used as an excuse to reduce on-call staffing.

#### Level 5 -- Adaptive
- **Required practices**:
  - Agents proactively identify anomalies and propose preventive actions before incidents occur.
  - Agent remediation accuracy is measured and used to expand or contract the approved action list.
  - Incident postmortems feed back into agent context and runbooks automatically.
- **Required artifacts**:
  - Anomaly detection integration with agent alerting.
  - Remediation accuracy metrics.
  - Automated postmortem-to-runbook feedback pipeline.
- **Anti-patterns**: Proactive alerts create noise without actionable recommendations. Feedback loops exist on paper but are never executed.

---

### Domain 6: Security and Compliance

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: No policy on AI agent usage exists. Sensitive data is inadvertently sent to third-party AI services. No awareness of data classification implications.

#### Level 1 -- Exploratory
- **Required practices**: Security team is aware that developers use AI agents. Informal guidance exists about not pasting secrets or PII into prompts.
- **Required artifacts**: Informal guidance (chat message, wiki note).
- **Anti-patterns**: Guidance is informal and inconsistent. No technical controls prevent data leakage to AI endpoints.

#### Level 2 -- Defined
- **Required practices**:
  - Acceptable use policy for AI agents is published (see `templates/policy-ai-agent-usage.md`).
  - Data classification rules define what can and cannot be sent to AI agent endpoints (e.g., no PII, no secrets, no classified data).
  - For regulated environments: AI agent traffic does not leave approved network boundaries (authorized endpoints only, or self-hosted models).
  - Secrets scanning runs on all agent-generated code before merge (e.g., gitleaks, truffleHog, GitLab Secret Detection, or platform-native scanning).
  - Agent API keys are managed centrally, not by individual developers.
- **Required artifacts**:
  - AI agent acceptable use policy.
  - Data classification matrix for AI agent usage.
  - Regulated environment network boundary documentation.
  - Secrets scanning configuration.
  - Centralized API key management (secrets management service such as HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or similar).
- **Anti-patterns**: Policy exists but is not enforced technically. Regulated and standard environments use the same AI endpoints. Developers use personal API keys.

#### Level 3 -- Managed
- **Required practices**:
  - All AI agent API calls are routed through an org-managed proxy or gateway that logs prompts and responses (with PII redaction).
  - Audit logs capture who used which agent, when, in which repo, and what was sent.
  - Regular (quarterly) review of agent usage logs for policy violations.
  - Supply chain security: agent-suggested dependencies are checked against an approved list or scanned for vulnerabilities before merge.
- **Required artifacts**:
  - API gateway or proxy with logging.
  - Audit log pipeline to your monitoring/SIEM system.
  - Quarterly review process documentation and findings.
  - Dependency allow-list or vulnerability scanning integration (e.g., Dependabot, Snyk, Artifactory Xray, Mend, or similar).
- **Anti-patterns**: Logging captures prompts containing sensitive data without redaction. Audit reviews are performed but findings are not remediated. Dependency scanning exists but agents suggest adding `--force` to bypass it.

#### Level 4 -- Optimized
- **Required practices**:
  - DLP (data loss prevention) controls actively prevent sensitive data from reaching AI agent endpoints.
  - Compliance checks are automated: agent-generated code is scanned for violations of applicable regulatory controls.
  - Agent access is governed by least-privilege roles scoped to specific repos, actions, and environments.
  - Exception process exists for cases where agents need access to restricted data or systems, with documented approval and time-bound access.
- **Required artifacts**:
  - DLP integration configuration.
  - Automated compliance scanning pipeline.
  - Access role definitions for agent access (per-repo or per-team).
  - Exception request template and approval log.
- **Anti-patterns**: DLP blocks legitimate agent usage with too many false positives. Compliance scanning produces reports nobody reads. Exception process is so burdensome that teams bypass it.

#### Level 5 -- Adaptive
- **Required practices**:
  - Security controls for agent usage are continuously tuned based on threat intelligence, audit findings, and false-positive rates.
  - Agent behavior is monitored for anomalies (unusual volume, unexpected repos, off-hours usage).
  - The org contributes to industry standards and shares anonymized learnings about AI agent security.
- **Required artifacts**:
  - Security control tuning log.
  - Behavioral anomaly detection for agent usage.
  - Industry engagement documentation.
- **Anti-patterns**: Controls are set once and never revisited. Anomaly detection generates alerts that are ignored. The org treats AI agent security as a solved problem.

---

### Domain 7: Knowledge Management

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: Documentation is sparse or outdated. Agents have no useful context to work with.

#### Level 1 -- Exploratory
- **Required practices**: Developers occasionally point agents at README files or code comments for context.
- **Required artifacts**: None required.
- **Anti-patterns**: Agents generate documentation that contradicts existing docs. No single source of truth exists.

#### Level 2 -- Defined
- **Required practices**:
  - Every repo has project-level agent instructions (e.g., CLAUDE.md or equivalent) that describe the project, conventions, architecture, and key decisions.
  - ADRs (Architecture Decision Records) are maintained and agents are instructed to reference them.
  - Agents are explicitly told where documentation lives and how to reference it.
- **Required artifacts**:
  - Agent instructions per repo (or monorepo section).
  - ADR directory with numbered records.
  - Agent context configuration pointing to docs.
- **Anti-patterns**: Agent instruction files are boilerplate copied between repos. ADRs exist but are never updated. Agents are given too much context (entire repo) instead of curated documents.

#### Level 3 -- Managed
- **Required practices**:
  - Documentation quality is measured (coverage, freshness, developer satisfaction).
  - Agents assist in maintaining documentation: updating READMEs when code changes, generating API docs, flagging stale docs.
  - A documentation index exists that agents can search (not just a flat file dump).
- **Required artifacts**:
  - Documentation quality metrics.
  - Agent-assisted doc maintenance workflow.
  - Searchable documentation index.
- **Anti-patterns**: Agents generate docs that are never reviewed and contain hallucinated content. Documentation index is out of date.

#### Level 4 -- Optimized
- **Required practices**:
  - Agents have access to a curated knowledge base via retrieval-augmented generation (RAG) or similar mechanism.
  - Knowledge base includes: org standards, approved patterns, past incident learnings, architectural guidelines.
  - Agent-generated documentation is reviewed and merged into the canonical knowledge base.
- **Required artifacts**:
  - RAG pipeline or knowledge retrieval system.
  - Curated knowledge base with editorial process.
  - Documentation merge workflow.
- **Anti-patterns**: RAG retrieves irrelevant or outdated content. Knowledge base has no editorial process and accumulates noise. Agents cite internal docs that were already deprecated.

#### Level 5 -- Adaptive
- **Required practices**:
  - Knowledge base is continuously refined based on agent usage patterns and developer feedback.
  - Agents identify knowledge gaps (questions they cannot answer from available context) and flag them for documentation.
  - Cross-team knowledge sharing is facilitated by agents that can access and synthesize documentation from multiple teams.
- **Required artifacts**:
  - Knowledge gap tracking system.
  - Cross-team knowledge access policies.
  - Knowledge base refinement changelog.
- **Anti-patterns**: Knowledge gaps are flagged but never addressed. Cross-team access violates data classification boundaries.

---

### Domain 8: Governance

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: No one owns AI agent strategy. Adoption is ungoverned.

#### Level 1 -- Exploratory
- **Required practices**: At least one person or team is informally tracking AI agent usage and interest across the org.
- **Required artifacts**: Informal tracking (spreadsheet, chat channel).
- **Anti-patterns**: Multiple teams evaluate agents independently with no coordination. Shadow IT risk.

#### Level 2 -- Defined
- **Required practices**:
  - An AI agent governance body exists (can be a working group, not necessarily a committee).
  - The governance body maintains the approved tools list, acceptable use policy, and exception process.
  - Agent usage decisions (new tools, expanded access, policy changes) go through a documented process.
  - Regulated-environment-specific governance requirements are documented and enforced separately.
- **Required artifacts**:
  - Governance body charter or working group terms of reference.
  - Decision log for agent-related decisions.
  - Regulated environment governance addendum.
- **Anti-patterns**: Governance body exists but has no authority. Decisions are made but not communicated. Regulated and standard governance are identical despite different requirements.

#### Level 3 -- Managed
- **Required practices**:
  - Agent usage is reported to leadership regularly (monthly or quarterly).
  - Cost of agent usage is tracked and allocated to teams or projects.
  - Risk register includes AI agent-specific risks.
  - Vendor management process covers AI agent providers.
- **Required artifacts**:
  - Leadership report template.
  - Cost allocation dashboard.
  - Risk register entries for AI agents.
  - Vendor assessment for AI agent providers.
- **Anti-patterns**: Reports are produced but leadership does not read them. Cost tracking is inaccurate. Risk register entries are generic and not actionable.

#### Level 4 -- Optimized
- **Required practices**:
  - Governance is embedded in tooling: policy-as-code enforces agent boundaries, approval workflows are automated, audit trails are complete.
  - Cross-functional governance includes engineering, security, legal, and compliance representatives.
  - Agent governance aligns with broader IT and data governance frameworks.
- **Required artifacts**:
  - Policy-as-code for agent governance.
  - Cross-functional governance meeting minutes or decision log.
  - Governance framework alignment documentation.
- **Anti-patterns**: Automated governance is brittle and blocks legitimate work. Cross-functional meetings happen but produce no decisions. Governance overhead discourages agent adoption.

#### Level 5 -- Adaptive
- **Required practices**:
  - Governance processes are continuously improved based on measured outcomes (adoption rate, incident rate, compliance findings).
  - The org anticipates regulatory changes related to AI and proactively adjusts policies.
  - Governance decisions are transparent and communicated org-wide.
- **Required artifacts**:
  - Governance improvement log.
  - Regulatory horizon scanning process.
  - Org-wide governance communications.
- **Anti-patterns**: Governance becomes self-serving and slows innovation without reducing risk. Regulatory scanning is reactive, not proactive.

---

### Domain 9: Evaluation and Measurement

#### Level 0 -- Unaware
- **Required practices**: None.
- **Required artifacts**: None.
- **Anti-patterns**: No data on agent usage or impact.

#### Level 1 -- Exploratory
- **Required practices**: Anecdotal feedback from developers about agent usefulness.
- **Required artifacts**: None required.
- **Anti-patterns**: Decisions about agent adoption are based on vendor claims or hype, not evidence.

#### Level 2 -- Defined
- **Required practices**:
  - At least 5 KPIs are defined for agent adoption (see KPIs section below).
  - Baseline measurements are taken before expanding agent usage.
  - Developer surveys capture sentiment about agent tools.
- **Required artifacts**:
  - KPI definitions document.
  - Baseline measurement report.
  - Survey instrument and initial results.
- **Anti-patterns**: KPIs are defined but not measured. Baselines are not taken, making it impossible to measure improvement. Surveys ask leading questions.

#### Level 3 -- Managed
- **Required practices**:
  - KPIs are measured regularly (monthly minimum).
  - A/B or cohort comparisons measure agent impact (e.g., teams with agents vs. teams without).
  - Agent quality is measured: defect rate in agent-generated code, test coverage of agent-generated code, rework rate.
- **Required artifacts**:
  - Monthly KPI report.
  - Cohort comparison analysis.
  - Agent code quality report.
- **Anti-patterns**: Metrics show no improvement but agent usage continues to expand. Cohort comparisons are not controlled for team differences. Quality metrics are gamed.

#### Level 4 -- Optimized
- **Required practices**:
  - All 12+ KPIs are measured and reviewed.
  - ROI analysis includes direct costs (API usage, tooling licenses), indirect costs (review overhead, incident rate changes), and benefits (velocity, quality, developer satisfaction).
  - Measurement drives decisions: tools that do not demonstrate value are deprecated; successful patterns are scaled.
- **Required artifacts**:
  - Full KPI dashboard.
  - ROI analysis report.
  - Decision log showing measurement-driven actions.
- **Anti-patterns**: ROI analysis is manipulated to justify predetermined conclusions. Sunk cost fallacy keeps underperforming tools alive. Metrics overhead exceeds the value of measurement.

#### Level 5 -- Adaptive
- **Required practices**:
  - Measurement framework itself is evaluated and improved.
  - Leading indicators (not just lagging) are tracked to predict agent impact.
  - The org shares measurement methodology and anonymized results with the industry.
- **Required artifacts**:
  - Measurement framework review log.
  - Leading indicator definitions and tracking.
  - Published case studies or anonymized reports.
- **Anti-patterns**: Measurement becomes an end in itself. The org over-indexes on metrics and loses sight of qualitative developer experience.

---

## Key Performance Indicators (KPIs)

| # | KPI | How to Measure | Target Direction |
|---|-----|---------------|-----------------|
| 1 | Agent adoption rate | Percentage of developers actively using approved agent tools (weekly active users / total developers) | Increase |
| 2 | Code acceptance rate | Percentage of agent suggestions accepted by developers (from telemetry) | Monitor (not purely "higher is better") |
| 3 | PR merge time (agent vs. human) | Median time from PR open to merge, segmented by agent-generated vs. human-written | Parity or better |
| 4 | Defect escape rate (agent vs. human) | Production defects per 1000 lines of code, segmented by origin | Parity or better |
| 5 | Agent-generated code rework rate | Percentage of agent-generated PRs that require follow-up fixes within 14 days | Decrease |
| 6 | CI pipeline time impact | Additional pipeline time added by agent CI steps | Minimize |
| 7 | Security finding rate (agent code) | Security vulnerabilities per 1000 lines in agent-generated code (from SAST/DAST) | Decrease |
| 8 | Cost per agent interaction | Total agent API spend / total agent interactions | Monitor |
| 9 | Developer satisfaction (agent tools) | Survey score (1-5) on agent tool usefulness, trust, and workflow integration | Increase |
| 10 | Time saved per developer per week | Self-reported or measured time saved by using agent tools (survey + telemetry) | Increase |
| 11 | Policy compliance rate | Percentage of agent usage that complies with acceptable use policy (from audit logs) | Target: >99% |
| 12 | Incident contribution score | For incidents where agents assisted: did agent reduce time-to-mitigation vs. similar past incidents? | Improve |
| 13 | IaC policy violation rate | Percentage of agent-generated IaC that fails policy-as-code checks | Decrease |
| 14 | Documentation freshness | Percentage of agent-maintained docs updated within last 90 days | Increase |
| 15 | Knowledge gap closure rate | Number of identified knowledge gaps resolved per quarter | Increase |

---

## High-Risk Change Categories Requiring Human Review

These categories of change, when generated or proposed by an AI agent, **must** receive explicit human review and approval before merge or execution. This requirement applies regardless of maturity level.

| # | Category | Why It Is High Risk | Required Review |
|---|----------|-------------------|----------------|
| 1 | Identity and access policy changes | Overly permissive policies can expose entire cloud accounts or subscriptions | Security team + infrastructure team |
| 2 | Network security changes | Security group, firewall rule, or network ACL changes can expose internal services | Security team |
| 3 | Database schema migrations | Data loss, corruption, or performance degradation risk | DBA or data team + application team |
| 4 | CI/CD pipeline modifications | Compromised pipelines can deploy malicious code or leak secrets | Platform team |
| 5 | Secrets or credential handling | Hardcoded secrets, weak encryption, or improper key management | Security team |
| 6 | Authentication/authorization logic | Broken auth is a top vulnerability category (OWASP) | Security team + application team |
| 7 | Production infrastructure changes | IaC changes to production environments can cause outages | Infrastructure team + change advisory |
| 8 | Regulated environment changes | Compliance violations (FedRAMP, HIPAA, PCI-DSS, GDPR, etc.) can have legal consequences | Compliance team + security team |
| 9 | Dependency additions or major version upgrades | Supply chain risk, breaking changes, license compliance | Application team lead |
| 10 | Data pipeline or ETL changes | Data integrity, PII exposure, compliance implications | Data team + compliance |

---

## Failure Conditions and Common Pitfalls

### Failure Condition 1: Uncontrolled Shadow Adoption
**Description**: Developers adopt AI agents outside of IT and security visibility, using personal accounts and unapproved tools.
**Mitigation**: Publish an acceptable use policy early (Level 2). Provide approved tools that are genuinely useful, so developers do not seek alternatives. Monitor network traffic for known AI service endpoints.

### Failure Condition 2: Checkbox Compliance
**Description**: The org creates policies and checklists but does not enforce them technically. Compliance is aspirational.
**Mitigation**: Enforce policies through automation (code ownership rules, CI gates, API gateways, DLP). Audit regularly and act on findings.

### Failure Condition 3: Agent-Generated Technical Debt
**Description**: Agents produce code that works but is poorly structured, over-abstracted, or inconsistent with org conventions, creating long-term maintenance burden.
**Mitigation**: Provide agents with strong project context (agent instruction files, style guides, approved patterns). Measure rework rate. Train reviewers to catch agent-specific code smells.

### Failure Condition 4: Review Theater
**Description**: Agent-generated PRs receive perfunctory reviews because reviewers assume the agent "knows what it is doing" or because the diff is too large to review carefully.
**Mitigation**: Enforce maximum PR size. Auto-label agent PRs. Provide reviewer training specific to agent-generated code. Track review depth metrics.

### Failure Condition 5: Security Blind Spots in Regulated Environments
**Description**: AI agent usage violates regulatory requirements because the org treats regulated environments the same as standard ones.
**Mitigation**: Maintain separate policies, network boundaries, and approved tool lists for regulated environments. Involve compliance early in agent adoption decisions.

### Failure Condition 6: Cost Surprise
**Description**: Agent API costs grow faster than expected because usage is not metered or allocated.
**Mitigation**: Centralize API key management. Set per-team or per-project usage quotas. Track cost per interaction. Review costs monthly.

### Failure Condition 7: Context Poisoning
**Description**: Agents produce incorrect output because they consume stale, contradictory, or irrelevant context from documentation or codebase.
**Mitigation**: Curate agent context deliberately. Maintain documentation freshness. Use RAG with editorial controls. Test agent outputs against known-good baselines.

### Failure Condition 8: Skill Atrophy
**Description**: Developers lose core engineering skills because they rely on agents for tasks they should understand deeply (debugging, architecture, security analysis).
**Mitigation**: Maintain requirements for human-authored work in critical areas. Rotate agent usage across tasks. Include skill maintenance in professional development plans.

### Failure Condition 9: Vendor Lock-in
**Description**: The org builds critical workflows around a single agent vendor's proprietary features, making switching costly.
**Mitigation**: Use vendor-neutral configurations where possible. Abstract agent interactions behind internal interfaces. Evaluate multiple vendors periodically.

### Failure Condition 10: Metrics Without Action
**Description**: The org collects extensive metrics on agent usage but never uses them to make decisions.
**Mitigation**: Assign owners to each KPI. Define thresholds that trigger action. Review metrics in existing forums (sprint retros, quarterly planning). Kill metrics that do not drive decisions.

---

## Anti-Pattern Summary

| # | Anti-Pattern | Domain(s) | Mitigation |
|---|-------------|-----------|-----------|
| 1 | Rubber-stamping agent PRs | PR/Review | Enforce review depth; auto-label agent PRs; limit PR size |
| 2 | Wildcard IAM/RBAC in agent-generated IaC | IaC, Security | Policy-as-code blocks wildcard policies; agent instructions prohibit them explicitly |
| 3 | Agents with production credentials | Ops, Security | Least-privilege roles; read-only for ops; pre-approved action lists |
| 4 | Personal API keys for agents | Security, Governance | Centralized key management; block personal key usage via network controls |
| 5 | Ignoring regulated/standard environment boundary | Security, IaC, Governance | Separate policies, tooling, and network paths for regulated environments |
| 6 | Agent-generated docs accepted without review | Knowledge | Review workflow for agent docs; hallucination checks |
| 7 | Bespoke CI pipelines per repo | CI/CD | Reusable pipeline component library; agents instructed to use shared components |
| 8 | Measuring agent adoption without quality | Evaluation | Track defect rate and rework rate alongside adoption rate |
| 9 | Governance without enforcement | Governance | Policy-as-code; automated compliance checks; regular audits |
| 10 | Context overload (sending entire repos to agents) | Dev Workflow, Knowledge | Curated context files; agent instructions; RAG with relevance filtering |
| 11 | Treating all agent output as equal quality | PR/Review, Evaluation | Segment metrics by agent vs. human; calibrate review rigor by risk |
| 12 | No rollback plan for agent-generated changes | CI/CD, IaC, Ops | Standard rollback procedures apply; tag agent-generated changes for easy identification |

---

## Definition of Done for Agent-Generated Work

Agent-generated work (code, IaC, documentation, configuration) is considered "done" when all of the following criteria are met:

1. **Functionally correct**: The change does what it is supposed to do, verified by tests (unit, integration, or manual, as appropriate).
2. **Tests included**: New functionality has corresponding tests. Bug fixes have regression tests. Test coverage does not decrease.
3. **Linted and formatted**: The change passes all configured linters and formatters without suppression of new warnings.
4. **Security scanned**: The change passes secrets scanning, SAST, and dependency vulnerability checks.
5. **Policy compliant**: The change passes all policy-as-code checks (OPA/Sentinel for IaC, org linting rules for code).
6. **Reviewed by a human**: At least one qualified human has reviewed the diff, not just the PR description.
7. **Context documented**: The PR description includes what the agent was asked to do, what it changed, and any limitations or follow-up items.
8. **Labeled**: The PR or commit is labeled or tagged as agent-generated or agent-assisted.
9. **No high-risk changes unaddressed**: If the change touches a high-risk category, the required additional reviewers have approved.
10. **Regulatory compliance verified** (if applicable): Changes to regulated environment resources have been verified against applicable compliance controls.
