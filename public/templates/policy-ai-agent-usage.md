# Policy: AI Agent Acceptable Use

**Effective Date**: [DATE]
**Owner**: [AI Agent Working Group / Engineering Leadership]
**Review Cycle**: Quarterly
**Classification**: Internal

---

## 1. Purpose

This policy defines the acceptable use of AI coding agents within your organization. It establishes rules for which tools may be used, how data is handled, what approvals are required, and how exceptions are managed. The policy applies to all employees, contractors, and vendors who write, review, or deploy code on behalf of the organization.

---

## 2. Scope

This policy covers:
- AI coding agents used in IDEs and development environments (e.g., GitHub Copilot, Claude Code, Cursor, Codex)
- AI agents used in CI/CD pipelines
- AI agents used during operational tasks and incident response
- Any tool that sends source code, documentation, or other organizational data to an AI model for code generation, analysis, or transformation

This policy does **not** cover:
- General-purpose AI assistants used for non-coding tasks (covered by separate policy)
- AI/ML models deployed as part of the organization's products (covered by ML model governance)

---

## 3. Approved Tools

Only AI agent tools on the approved list may be used for work on organizational code and systems.

**Current approved tools**:

| Tool | Approved Version(s) | Approved Endpoint | Notes |
|------|---------------------|-------------------|-------|
| [Tool 1, e.g., Claude Code] | [Version] | [Org proxy, cloud model API, or direct vendor API] | Primary agent tool |
| [Tool 2, e.g., GitHub Copilot] | [Version] | [Org-managed license endpoint] | IDE completion |
| [Add more as approved] | | | |

**Unapproved tools**: Any AI coding tool not on the approved list is prohibited for use on organizational code. This includes:
- Free-tier or personal accounts for approved tools (use org-managed accounts only)
- Web-based AI chat interfaces (ChatGPT web, Claude.ai web) for pasting org code
- Third-party IDE plugins that send code to unapproved endpoints
- Self-hosted or open-source models not vetted by the security team

**How to request approval for a new tool**: Submit a request to the AI Agent Working Group using the exception process in Section 9.

---

## 4. Data Handling Rules

### 4.1 Data Classification for Agent Interactions

| Data Classification | May Be Sent to Approved Agent Endpoints | Additional Controls |
|--------------------|-----------------------------------------|-------------------|
| Public | Yes | None |
| Internal | Yes | Logged via org API proxy |
| Confidential | Only with DLP controls active (Level 4+). Before DLP: No. | DLP scan before transmission. Prompt/response logged and reviewed. |
| Restricted (PII, PHI, secrets) | No | Never send to any AI agent endpoint. Technical controls must prevent this. |
| Classified / regulated data | No | Never send to any AI agent endpoint. Network boundaries enforced per applicable regulation. |

### 4.2 Specific Prohibitions

The following must **never** be included in prompts, code context, or files accessible to AI agents:
- API keys, tokens, passwords, or other credentials
- Personally identifiable information (PII): names, emails, SSNs, addresses linked to real individuals
- Protected health information (PHI)
- Controlled or classified data as defined by your applicable regulatory framework
- Customer data from production databases
- Encryption keys or key material
- Security vulnerability details not yet disclosed or patched

### 4.3 Regulated Environment Data Handling

For teams working in regulated environments (e.g., FedRAMP, HIPAA, PCI-DSS, GDPR):
- All AI agent API traffic must route through approved endpoints only. Consult your compliance team for the list of approved services and regions.
- No organizational data from regulated systems may be sent to unapproved AI endpoints.
- Data residency requirements apply: AI model inference must occur within approved regions or boundaries.
- Logging of agent interactions must comply with applicable audit and accountability controls (e.g., NIST 800-53 AU controls, HIPAA audit requirements, GDPR record-keeping).

---

## 5. Authentication and Access

### 5.1 API Key Management

- AI agent API keys and tokens are managed centrally by the platform team.
- Keys are stored in your organization's secrets management service (e.g., HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager).
- Developers do not have direct access to API keys. Keys are injected into dev environments and CI environments at runtime.
- Personal API keys for approved tools are prohibited for organizational work.

### 5.2 Developer Access

- Access to AI agent tools is granted by team, not individually.
- Access is provisioned through the standard onboarding process.
- Offboarding includes revocation of agent tool access.
- Contractors and vendors receive access only for the duration of their engagement and only with explicit approval.

### 5.3 Agent Access to Systems

- Agents in IDE/dev environment: access to local project workspace only. No access to other projects, host filesystem, or credential stores.
- Agents in CI: read access to repo checkout. Write access limited to PR/MR comments. No deployment credentials.
- Agents in ops: read-only access to logs and metrics via scoped access role. Controlled write mode requires Level 4 maturity and IC approval (see runbook-agent-safety.md).

---

## 6. Code Attribution and Labeling

### 6.1 Attribution

All code generated with AI agent assistance must be attributed:
- Commits include a `Co-Authored-By` trailer identifying the agent (e.g., `Co-Authored-By: Claude <noreply@anthropic.com>`).
- PRs that are substantially agent-generated or agent-assisted must be labeled (e.g., `agent-assisted` label).

### 6.2 Intellectual Property

- Agent-generated code is treated as work product of the organization, subject to the same IP policies as human-written code.
- Developers are responsible for reviewing agent-generated code and ensuring it does not introduce copyrighted material, license violations, or patent-infringing implementations.
- If an agent generates code that closely resembles publicly available open-source code, the developer must verify license compatibility before merging.

---

## 7. Review and Quality Requirements

### 7.1 Human Review

All agent-generated code must be reviewed by at least one qualified human before merging. "Qualified" means the reviewer:
- Has write access to the repository.
- Understands the codebase and the change being made.
- Has completed agent-aware review training (when available).

### 7.2 Automated Checks

Agent-generated PRs must pass all automated quality gates before merge:
- Linting and formatting
- Unit and integration tests
- Secrets scanning
- SAST (Static Application Security Testing)
- Dependency vulnerability scanning
- Policy-as-code checks (for IaC)

### 7.3 High-Risk Changes

Agent-generated changes in high-risk categories (see `docs/ai-agent-cmm.md`, High-Risk Change Categories) require additional review as specified. The developer is responsible for identifying high-risk changes and requesting the appropriate reviewers.

---

## 8. Monitoring and Compliance

### 8.1 Logging

All AI agent API interactions are logged via the org API proxy. Logs include:
- Timestamp
- Developer or system identity
- Repository or project context
- Model used
- Token count
- Request and response hashes (full content in standard environments; hashes only in regulated environments, unless full logging is required by compliance)

### 8.2 Audit

- The security team reviews agent usage logs quarterly for policy violations.
- Findings are documented and remediated.
- Repeat violations result in escalation to the developer's manager and, if necessary, revocation of agent access.

### 8.3 Metrics

Agent usage metrics (adoption, quality, cost, compliance) are tracked as defined in the KPI framework (see `docs/ai-agent-cmm.md`). Monthly reports are provided to engineering leadership.

---

## 9. Exception Process

Exceptions to this policy (e.g., using an unapproved tool, sending confidential data to an agent endpoint, granting agents write access in ops) require:

1. **Written request**: Submit to the AI Agent Working Group via [CHANNEL/TOOL].
2. **Justification**: Explain why the exception is needed and what alternative approaches were considered.
3. **Risk assessment**: Describe the security, compliance, and operational risks of the exception.
4. **Mitigations**: Describe what additional controls will be in place during the exception.
5. **Duration**: Exceptions are time-bound (maximum 90 days, renewable).
6. **Approval**: Requires sign-off from:
   - AI Agent Working Group lead
   - Security team representative
   - Compliance team representative (for exceptions impacting regulated environments)

**Exception log**: All exceptions are logged in [LOCATION] and reviewed during quarterly audits.

---

## 10. Incident Response

If an AI agent is involved in a security incident (data leak, unauthorized access, generated vulnerable code that was exploited):

1. Follow the standard incident response process.
2. Immediately revoke the agent's API credentials.
3. Preserve all agent interaction logs.
4. Notify the AI Agent Working Group.
5. Include the agent's involvement in the incident postmortem.
6. Update this policy if the incident reveals a gap.

---

## 11. Training Requirements

All developers using AI agent tools must complete:
- **Before first use**: Basic training covering this policy, approved tools, data handling rules, and PR review requirements.
- **Within 90 days**: Agent-aware code review training.
- **Annually**: Policy refresher and updates on new tools, risks, and best practices.

Training completion is tracked by [HR/LEARNING_SYSTEM] and reported to team leads.

---

## 12. Policy Violations

| Violation Severity | Examples | Consequence |
|-------------------|----------|-------------|
| Minor | Forgetting to label an agent-assisted PR. Using an approved tool in an unapproved way (e.g., wrong endpoint). | Counseling. Training refresher. |
| Major | Using an unapproved tool for org code. Sending confidential data to an agent endpoint. | Agent access revoked for 30 days. Manager notified. Retraining required. |
| Critical | Sending restricted/classified data to an AI endpoint. Using agents to bypass security controls. Covering up agent-generated security vulnerabilities. | Agent access permanently revoked. HR and legal involvement. Potential disciplinary action. |

---

## 13. Policy Review and Updates

This policy is reviewed quarterly by the AI Agent Working Group. Updates are communicated to all affected employees via [COMMUNICATION_CHANNEL].

**Change log**:

| Date | Change | Author |
|------|--------|--------|
| [DATE] | Initial policy published | [AUTHOR] |
| | | |

---

## Appendix A: Regulated Environment Addendum

For teams operating in regulated environments, the following additional requirements apply. Adapt this section to the specific frameworks your organization must comply with.

### FedRAMP / FISMA (U.S. Federal)
1. **Authorized endpoints only**: AI agent inference must use FedRAMP-authorized services (e.g., AWS Bedrock in GovCloud, Azure Gov). No exceptions.
2. **Data residency**: All agent interaction data (prompts, responses, logs) must reside within approved GovCloud or government regions.
3. **Audit controls**: Agent interaction logging must meet NIST 800-53 AU-2, AU-3, and AU-6 requirements.
4. **Access control**: Agent access must comply with NIST 800-53 AC-2 and AC-6.
5. **System security plan**: AI agent tooling must be documented in the relevant SSP.
6. **Continuous monitoring**: Agent usage is included in the continuous monitoring program per NIST 800-53 CA-7.
7. **Incident response**: Agent-related incidents must be reported per Federal incident reporting timelines.

### HIPAA (Healthcare)
1. **No PHI in prompts**: AI agents must never process protected health information.
2. **BAA coverage**: If the AI service processes any data from systems that handle PHI, a Business Associate Agreement must be in place.
3. **Audit trail**: Agent interactions on systems in scope for HIPAA must have audit trails meeting HIPAA Security Rule requirements.
4. **Minimum necessary**: Agent access follows the minimum necessary standard.

### PCI-DSS (Payment Card)
1. **No cardholder data**: AI agents must never process cardholder data (PAN, CVV, etc.).
2. **Network segmentation**: Agents operating on systems in the cardholder data environment must use isolated, segmented network paths.
3. **Access control**: Agent access to CDE systems follows PCI-DSS Requirement 7 (restrict access by business need).
4. **Logging**: Agent interactions on CDE systems must meet PCI-DSS Requirement 10 (track and monitor all access).

### GDPR (EU Data Protection)
1. **No personal data in prompts**: AI agents must not process personal data as defined by GDPR unless a lawful basis exists and a DPIA has been completed.
2. **Data transfer**: If the AI inference endpoint is outside the EU/EEA, appropriate transfer mechanisms (SCCs, adequacy decisions) must be in place.
3. **Record-keeping**: Agent usage involving personal data must be documented per Article 30.
4. **Data subject rights**: Processes must exist to handle data subject requests for any data processed by agents.

### SOC 2
1. **Access controls**: Agent access is governed by organization-wide access control policies aligned with SOC 2 Trust Services Criteria (CC6).
2. **Monitoring**: Agent usage is included in the organization's monitoring controls per CC7.
3. **Change management**: Changes to agent tooling and configuration follow the organization's change management process per CC8.

This addendum supersedes the main policy where there is a conflict. If in doubt, apply the more restrictive rule.

---

## Appendix B: Quick Reference for Developers

```
AI AGENT POLICY - QUICK REFERENCE

APPROVED TOOLS ONLY:     Use only tools on the approved list with org-managed accounts.
NO PERSONAL KEYS:        Use org-provided credentials, not personal API keys.
NO SECRETS IN PROMPTS:   Never paste API keys, passwords, PII, or classified data into agent prompts.
LABEL YOUR WORK:         Tag agent-assisted commits and PRs.
HUMAN REVIEW REQUIRED:   Every agent-generated PR needs at least one human reviewer.
REGULATED IS DIFFERENT:  Regulated environments use approved endpoints only. Stricter rules apply.
REPORT PROBLEMS:         If an agent generates something concerning, tell your lead and security.
ASK QUESTIONS:           If you are unsure whether something is allowed, ask the Working Group.
```
