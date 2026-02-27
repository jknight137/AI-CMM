# Runbook: Agent Safety Rules for Production and Incident Work

## Purpose

This runbook defines the do-no-harm rules for using AI agents during production operations and incident response. All on-call engineers, SREs, and incident commanders must follow these rules.

---

## Rule 1: Agents Are Advisory by Default

Agents provide suggestions, analysis, and drafts. They do not execute actions on production systems unless explicitly operating under the Controlled Write Mode described in Rule 6.

**What this means in practice**:
- Read agent output. Evaluate it. Then decide whether to act.
- Never copy-paste an agent-suggested command into a production terminal without understanding it.
- If the agent suggests something you do not understand, ask the agent to explain it, then verify independently.

---

## Rule 2: Read-Only Access Only (Default)

By default, agents used during ops and incident response have **read-only** access to production systems.

**Permitted read-only actions**:
- Search CloudWatch Logs
- Query CloudWatch Metrics
- Read ECS service/task status
- Read load balancer target health
- Search past incident records
- Read application configuration (non-secrets)
- Read Terraform state (read-only backend)

**Prohibited without Controlled Write Mode**:
- Restart, stop, or scale any service
- Modify any configuration
- Deploy any code
- Change any IAM role, security group, or network configuration
- Toggle any feature flag
- Run any database query (even SELECT, unless explicitly approved for the incident)
- Access or read any secrets

---

## Rule 3: No Production Credentials for Agents

Agents must never receive:
- Production AWS access keys or session tokens (beyond the scoped read-only role)
- Database connection strings or credentials
- API keys for production third-party services
- SSH keys to production instances
- Kubernetes/ECS exec access (even if Kubernetes is adopted later)

**How credentials are managed**:
- Agents use a dedicated IAM role with read-only permissions, assumed via role chaining.
- The role is scoped to specific log groups, metric namespaces, and services relevant to the incident.
- The role has a session duration limit (1 hour maximum, renewable by the IC).

---

## Rule 4: Incident Commander Controls Agent Engagement

The incident commander (IC) decides:
- Whether to engage agent assistance at all.
- Which agent capabilities to enable (log search, timeline building, postmortem drafting, etc.).
- Whether to escalate to Controlled Write Mode (Level 4+ only).
- When to disengage the agent.

**No one other than the IC authorizes agent actions during an active incident.**

If there is no designated IC, agent usage defaults to read-only log search only.

---

## Rule 5: All Agent Actions Are Logged

Every agent interaction during an incident must be logged to:
1. The incident Slack channel (or equivalent communication channel).
2. The incident audit trail (ticketing system or dedicated log).

**What to log**:
- Timestamp
- Who initiated the agent action
- What the agent was asked to do
- What the agent responded or did
- Whether the response was acted on by a human

**Logging format** (paste into incident channel):
```
[TIMESTAMP] AGENT ACTION
Initiated by: [NAME]
Request: [WHAT_WAS_ASKED]
Response: [SUMMARY_OF_AGENT_OUTPUT]
Action taken: [WHAT_THE_HUMAN_DID_WITH_THE_OUTPUT]
```

---

## Rule 6: Controlled Write Mode (Level 4+ Only)

At maturity Level 4 and above, agents may execute a limited set of pre-approved remediation actions. This is called Controlled Write Mode.

### Prerequisites for Controlled Write Mode
- The organization has reached Level 4 in the Ops and Incident Response domain.
- The pre-approved action list has been reviewed and signed off by the SRE lead and security team.
- The agent has a dedicated IAM role for write actions, separate from the read-only role.
- Audit logging for write actions is verified and operational.

### Pre-Approved Action List (Example -- Customize for Your Org)

| Action | Parameters | Limits | Required Approver |
|--------|-----------|--------|------------------|
| Restart ECS service | Cluster, service name | One service at a time | IC |
| Scale ECS service task count | Cluster, service name, desired count | Max 2x current count | IC |
| Toggle feature flag | Flag name | Only flags on the approved list | IC |
| Create incident ticket | Title, description, severity | No limit | IC |
| Post to incident Slack channel | Message text | No limit | IC |
| Roll back to previous ECS task definition | Cluster, service name | One revision back only | IC + SRE lead |

### Actions NOT Permitted in Controlled Write Mode
- Any IAM or security group change
- Any database write or schema change
- Any Terraform apply
- Any deployment of new code
- Any change to networking (VPC, route tables, NACLs)
- Any action not on the pre-approved list

### Approval Flow for Controlled Write Mode
1. Agent proposes an action and displays it to the IC.
2. IC reviews the proposed action, parameters, and expected outcome.
3. IC explicitly approves by typing a confirmation command.
4. Agent executes the action.
5. Agent reports the result.
6. Action, approval, and result are logged to the audit trail.

If the action fails, the agent reports the failure and does not retry without a new IC approval.

---

## Rule 7: Federal Environment Restrictions

For incidents affecting Federal environments:
- Agents are restricted to **read-only mode only** until a FedRAMP assessment explicitly authorizes Controlled Write Mode for agent-assisted remediation.
- All agent interactions during Federal incidents must be logged to the GovCloud audit trail.
- Agent API calls must route through GovCloud endpoints only.
- No incident data from Federal environments may be sent to commercial AI endpoints.
- The IC must verify that the agent is operating in Federal mode before engagement.

---

## Rule 8: Agent Output Verification

Before acting on any agent output during an incident:

1. **Cross-reference**: Verify agent claims against at least one independent source (logs, metrics, dashboard).
2. **Challenge assumptions**: If the agent suggests a root cause, ask "what evidence supports this?" and "what would disprove this?"
3. **Scope check**: Ensure the agent's suggestion addresses the actual incident, not a related but different issue.
4. **Blast radius check**: Before executing any remediation, confirm the blast radius is acceptable (which services, how many users, can it be rolled back).

---

## Rule 9: Disengagement Criteria

Stop using agent assistance and rely on human-only response when:
- The agent provides demonstrably incorrect information twice in a row.
- The incident involves data classified above the agent's access level.
- The incident involves a suspected security breach (agents may be compromised or may leak information).
- The IC determines that agent output is slowing down rather than accelerating response.
- The agent's session has timed out and the incident is in a critical phase (re-authorization can wait).

---

## Rule 10: Post-Incident Review of Agent Usage

Every incident where agents were used must include a section in the postmortem:

1. **Was agent assistance helpful?** (Yes / Partially / No)
2. **What did the agent do well?** (Specific examples)
3. **What did the agent get wrong?** (Specific examples)
4. **Did agent usage speed up or slow down the response?** (Estimate in minutes)
5. **Were any safety rules violated?** (If yes, what happened and what corrective action is needed)
6. **Should the pre-approved action list be updated?** (Add or remove actions based on experience)

This data feeds into the Evaluation and Measurement domain (KPI #12: Incident Contribution Score).

---

## Quick Reference Card

Print or bookmark this for on-call use:

```
AI AGENT INCIDENT RULES - QUICK REFERENCE

1. Agents advise. Humans decide and act.
2. Read-only access by default. No production credentials.
3. IC controls all agent engagement.
4. Log every agent interaction to the incident channel.
5. Verify agent output before acting on it.
6. Controlled Write Mode (Level 4+): pre-approved actions only, IC approval required.
7. Federal incidents: read-only only. GovCloud endpoints only.
8. Disengage if agent is wrong twice, incident is security-related, or agent slows you down.
```
