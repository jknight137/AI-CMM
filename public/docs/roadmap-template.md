# AI Agent Adoption Roadmap

## How to Use This Template

1. Complete the assessment rubric (see `docs/assessment-rubric.md`) to determine your current maturity level.
2. Set a target maturity level for each domain (recommend targeting current + 1 for the first cycle).
3. Use the 90-day plan to establish foundations and run initial pilots.
4. Use the 12-month plan for sustained adoption and optimization.
5. Customize milestones and dates to your organization's planning cycle.

---

## 90-Day Plan (Foundation and Pilot)

### Objective
Establish policy foundations, set up approved tooling, run a controlled pilot with 2-3 teams, and collect baseline metrics.

### Week 1-2: Governance and Policy

| Action | Owner | Deliverable | Done |
|--------|-------|-------------|------|
| Form AI Agent Working Group (engineering, security, platform, compliance) | Engineering leadership | Working group charter with named members | [ ] |
| Draft acceptable use policy using `templates/policy-ai-agent-usage.md` | Working group | Policy document in review | [ ] |
| Define data classification rules for agent interactions | Security team | Data classification matrix | [ ] |
| Identify regulated-environment-specific restrictions and document separately | Compliance team | Regulated environment addendum to policy | [ ] |
| Select approved agent tools (IDE agents, CLI agents) | Working group | Approved tools list | [ ] |

### Week 2-4: Tooling and Infrastructure

| Action | Owner | Deliverable | Done |
|--------|-------|-------------|------|
| Configure org-managed API endpoints (e.g., cloud-hosted model API, org proxy, or direct vendor API with centralized keys) | Platform team | API gateway/proxy running in dev | [ ] |
| Set up centralized API key management (e.g., HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager) | Platform team | Keys provisioned for pilot teams | [ ] |
| Build dev environment image with approved agent tooling | Platform team | Image published to artifact registry | [ ] |
| Configure secrets scanning on pilot repos (e.g., gitleaks, truffleHog, or platform-native secret scanning) | Security team | Scanning active on pilot repos | [ ] |
| Set up code ownership rules for CI/CD config files in pilot repos | Platform team | Code ownership files committed | [ ] |
| Create reusable CI pipeline components for agent gates (lint, test, secrets scan) | Platform team | Components published to shared repo or registry | [ ] |

### Week 3-5: Context and Standards

| Action | Owner | Deliverable | Done |
|--------|-------|-------------|------|
| Write agent instruction templates for pilot project types (service, library, IaC module) | Tech leads + platform team | Templates in shared repo | [ ] |
| Add agent instruction files to each pilot repo, customized for that project | Pilot team developers | Instruction files committed in each pilot repo | [ ] |
| Create PR/MR template with agent disclosure field | Platform team | PR template deployed to pilot repos | [ ] |
| Write PR checklist with agent-specific items (use `templates/pr-checklist.md`) | Working group | Checklist deployed to pilot repos | [ ] |
| Document IaC conventions and guardrails if IaC is in pilot scope | Infrastructure team | IaC style guide | [ ] |

### Week 4-6: Pilot Launch

| Action | Owner | Deliverable | Done |
|--------|-------|-------------|------|
| Select 2-3 pilot teams (mix of seniority, project types) | Working group | Pilot team list | [ ] |
| Conduct pilot kickoff: training on approved tools, policy, expectations | Platform team | Training session completed | [ ] |
| Distribute runbook safety rules if ops is in pilot scope | SRE/platform team | Runbook distributed | [ ] |
| Begin collecting baseline metrics (pre-agent) for pilot teams | Platform team | Baseline report | [ ] |
| Pilot teams begin using agents in daily work | Pilot teams | Usage starts | [ ] |

### Week 6-10: Observation and Measurement

| Action | Owner | Deliverable | Done |
|--------|-------|-------------|------|
| Collect telemetry on agent usage (interactions, acceptances, repos) | Platform team | Telemetry pipeline active | [ ] |
| Track PR metrics for pilot teams (merge time, defect rate, rework rate) | Platform team | Metrics collection active | [ ] |
| Conduct mid-pilot developer survey (satisfaction, friction, concerns) | Working group | Survey results | [ ] |
| Review audit logs for policy compliance | Security team | Audit review report | [ ] |
| Review agent-generated PRs for quality patterns | Tech leads | Quality observations documented | [ ] |

### Week 10-13: Evaluation and Next Steps

| Action | Owner | Deliverable | Done |
|--------|-------|-------------|------|
| Compile pilot results: metrics, survey, audit findings, quality observations | Working group | Pilot evaluation report | [ ] |
| Identify what worked, what did not, and what needs to change | Working group | Lessons learned document | [ ] |
| Decide: expand, adjust, or pause agent rollout | Engineering leadership | Go/no-go decision documented | [ ] |
| Update policy and tooling based on pilot findings | Working group | Updated policy and configs | [ ] |
| Present findings and recommendation to leadership | Working group | Leadership briefing | [ ] |

### 90-Day Milestone Checklist

- [ ] Acceptable use policy published and acknowledged by pilot teams
- [ ] Approved tools configured with org-managed API endpoints
- [ ] Dev environment images with agent tooling available in artifact registry
- [ ] Agent instruction files and PR templates deployed to pilot repos
- [ ] Secrets scanning active on all pilot repos
- [ ] Baseline metrics collected for pilot teams
- [ ] Pilot completed with 2-3 teams
- [ ] Pilot evaluation report delivered to leadership
- [ ] Go/no-go decision made for broader rollout

---

## 12-Month Plan (Scale and Optimize)

### Workstreams

The 12-month plan is organized into 6 parallel workstreams. Each workstream has a lead, quarterly milestones, and associated KPIs.

---

### Workstream 1: Tooling

**Lead**: Platform Engineering

**Objective**: Make approved agent tools available, reliable, and well-integrated for all development teams.

| Quarter | Milestone | KPI Target |
|---------|-----------|-----------|
| Q1 | Pilot tooling deployed (dev environments, API gateway, key management). Agent gates in CI for pilot repos. | Agent adoption rate: 30% of pilot team developers active weekly |
| Q2 | Tooling available to all teams. Dev environment images auto-updated via registry pipeline. Agent auto-labeling deployed org-wide. | Agent adoption rate: 25% of all developers active weekly |
| Q3 | Cross-repo context system (RAG or doc index) deployed. Agent CLI integrated into CI for test generation, doc updates. | Developers report agents have "good" or "great" context: >60% survey score |
| Q4 | Tooling is self-service. New repos get agent configs automatically via repo templates. Agents run in CI across standard pipeline templates. | Agent adoption rate: 50%+ of all developers active weekly |

---

### Workstream 2: Policy

**Lead**: Security and Compliance

**Objective**: Establish, enforce, and refine policies for safe and compliant agent usage across all environments.

| Quarter | Milestone | KPI Target |
|---------|-----------|-----------|
| Q1 | Acceptable use policy published. Data classification matrix defined. Regulated environment addendum written. API traffic routed through org proxy. | Policy compliance rate: >95% for pilot teams |
| Q2 | Policy enforced via technical controls (DLP, network boundaries, code ownership rules). Quarterly audit process established. Secrets scanning org-wide. | Policy compliance rate: >97% org-wide |
| Q3 | Exception process operational. Automated compliance scanning for agent-generated code. Regulated environment policy-as-code enforced. | Security finding rate in agent code: baseline established |
| Q4 | Policy tuned based on audit findings and false-positive rates. Behavioral anomaly detection for agent usage deployed. | Policy compliance rate: >99%. Security finding rate: decreasing trend |

---

### Workstream 3: Training

**Lead**: Engineering Leadership + Developer Experience

**Objective**: Ensure all developers and reviewers can use agents effectively and review agent-generated code critically.

| Quarter | Milestone | KPI Target |
|---------|-----------|-----------|
| Q1 | Pilot team training delivered (tools, policy, review, prompt engineering). Training materials created. | 100% of pilot team members trained |
| Q2 | Training available org-wide (self-paced + live sessions). Reviewer-specific training on agent code patterns. | 50% of developers completed training |
| Q3 | Advanced training: prompt engineering, agent instruction authoring, agent-in-CI usage. Lunch-and-learn series on agent best practices. | 70% of developers completed at least basic training |
| Q4 | Training updated based on quality data and developer feedback. Onboarding includes agent training for new hires. | 90% of developers completed training. Developer satisfaction with agents: >3.5/5 |

---

### Workstream 4: Pilot Teams

**Lead**: Engineering Leadership

**Objective**: Progressively expand agent usage from pilot teams to the full org, using evidence from each phase.

| Quarter | Milestone | KPI Target |
|---------|-----------|-----------|
| Q1 | 2-3 pilot teams running. Pilot evaluation completed. | Pilot teams demonstrate parity or improvement on defect rate and merge time |
| Q2 | Expand to 5-8 teams based on pilot findings. Include at least one regulated-environment team if applicable. Adjust tooling/policy based on pilot lessons. | Expanded teams onboarded within 2 weeks |
| Q3 | Agent usage available to all willing teams. Adoption is opt-in with support from platform team. | 50%+ of teams have at least one active agent user |
| Q4 | Agent usage is standard practice. Teams that do not use agents are the exception. | 75%+ of teams actively using agents |

---

### Workstream 5: Platform Enablement

**Lead**: Platform Engineering

**Objective**: Build platform capabilities that make agent usage safe, scalable, and measurable.

| Quarter | Milestone | KPI Target |
|---------|-----------|-----------|
| Q1 | API gateway with logging deployed. Dev environment images in artifact registry. CI agent gates for pilot repos. Centralized key management. | Infrastructure operational for pilot |
| Q2 | Telemetry pipeline collecting agent usage data. Cost tracking by team/project. PR metrics segmented by agent vs. human. | Telemetry covers >90% of agent interactions |
| Q3 | RAG or knowledge retrieval system. Policy-as-code for IaC. Automated compliance scanning. Risk scoring for PRs. | All 15 KPIs measured and reported monthly |
| Q4 | Self-service onboarding for new repos. Automated agent config deployment. Anomaly detection for agent usage. | Onboarding a new team takes <1 day of platform support |

---

### Workstream 6: Measurement

**Lead**: Engineering Leadership + Platform Engineering

**Objective**: Build a data-driven understanding of agent impact and use it to make decisions.

| Quarter | Milestone | KPI Target |
|---------|-----------|-----------|
| Q1 | 5 core KPIs defined and baselined: adoption rate, defect rate, merge time, developer satisfaction, policy compliance. | Baselines established for pilot teams |
| Q2 | Expand to 10+ KPIs. Cohort comparison (agent vs. non-agent teams). Monthly reporting to leadership. | Monthly report delivered on schedule |
| Q3 | All 15 KPIs active. ROI analysis completed. Measurement drives at least 2 concrete decisions (tool changes, policy updates, training focus). | ROI report delivered. 2+ measurement-driven decisions documented |
| Q4 | Leading indicators identified and tracked. Measurement framework reviewed and refined. Results shared internally (and externally if appropriate). | Measurement framework review completed. Annual report published |

---

## KPI Summary and Measurement Methods

| # | KPI | Measurement Method | Data Source | Frequency |
|---|-----|-------------------|-------------|-----------|
| 1 | Agent adoption rate | Weekly active users / total developers | Telemetry pipeline, HR headcount | Monthly |
| 2 | Code acceptance rate | Accepted suggestions / total suggestions | IDE telemetry (Copilot dashboard, Claude telemetry, or equivalent) | Monthly |
| 3 | PR merge time (agent vs. human) | Median PR open-to-merge, segmented by label | SCM API, PR metrics dashboard | Monthly |
| 4 | Defect escape rate (agent vs. human) | Production defects per 1K LOC, segmented by origin label | Issue tracker + PR labels + LOC analysis | Quarterly |
| 5 | Agent code rework rate | Agent PRs with follow-up fix within 14 days / total agent PRs | SCM API, PR labels | Monthly |
| 6 | CI pipeline time impact | Agent step duration / total pipeline duration | CI platform timing data | Monthly |
| 7 | Security finding rate (agent code) | SAST/DAST findings per 1K LOC in agent-generated code | SAST tool + PR labels + LOC analysis | Quarterly |
| 8 | Cost per agent interaction | Total API spend / total interactions | API gateway logs, billing data | Monthly |
| 9 | Developer satisfaction | Survey score (1-5) | Quarterly survey | Quarterly |
| 10 | Time saved per developer per week | Survey self-report + telemetry estimate | Survey + telemetry | Quarterly |
| 11 | Policy compliance rate | Compliant interactions / total interactions | Audit log analysis | Monthly |
| 12 | Incident contribution score | Time-to-mitigate with agent vs. baseline without | Incident tracker, agent action logs | Per-incident, aggregated quarterly |
| 13 | IaC policy violation rate | Failed policy checks / total agent IaC PRs | CI pipeline results + PR labels | Monthly |
| 14 | Documentation freshness | Agent-maintained docs updated in last 90 days / total agent-maintained docs | Git commit history on doc files | Quarterly |
| 15 | Knowledge gap closure rate | Gaps resolved this quarter / gaps identified this quarter | Knowledge gap tracker | Quarterly |

---

## Milestone Summary (12-Month View)

| Month | Key Milestones |
|-------|---------------|
| 1 | Working group formed. Policy drafted. Approved tools selected. |
| 2 | Tooling deployed for pilot. Dev environment images in registry. API gateway live. |
| 3 | Pilot launched with 2-3 teams. Baselines collected. Training delivered. |
| 4 | Mid-pilot review. Adjust tooling and policy. |
| 5 | Pilot evaluation complete. Go/no-go for expansion. |
| 6 | Expanded to 5-8 teams. Regulated-environment team included if applicable. Org-wide secrets scanning. |
| 7 | Telemetry and cost tracking operational. 10+ KPIs active. |
| 8 | Agent-in-CI deployed (test gen, doc updates). RAG system in progress. |
| 9 | ROI analysis complete. Policy-as-code for IaC. All KPIs measured. |
| 10 | Agent usage open to all teams. Self-service onboarding. Anomaly detection. |
| 11 | Training updated. Advanced training available. Measurement framework reviewed. |
| 12 | Annual report. Roadmap for year 2. Target: Level 3-4 across domains. |

---

## Risk Register for Roadmap Execution

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Pilot teams do not see value and abandon agents | Medium | High | Select teams working on tasks where agents add clear value (greenfield, test generation, doc heavy). Provide dedicated support. |
| Regulatory compliance review delays tooling rollout | High | Medium | Engage compliance in Week 1. Run regulatory approval in parallel with standard deployment. |
| API costs exceed budget | Medium | Medium | Set per-team quotas. Monitor daily in first month. Adjust model selection (use smaller models for simple tasks). |
| Security incident involving agent-generated code | Low | High | Secrets scanning, SAST, policy-as-code, and human review reduce risk. Incident response plan covers agent-related incidents. |
| Developer resistance to policy and oversight | Medium | Medium | Emphasize that policies protect developers. Make approved tools easy to use. Collect and act on feedback. |
| Platform team capacity insufficient | High | High | Prioritize ruthlessly. Automate early. Consider contractor or vendor support for initial buildout. |
| Vendor changes pricing or capabilities | Medium | Medium | Avoid deep vendor lock-in. Abstract agent interactions. Evaluate alternatives periodically. |
