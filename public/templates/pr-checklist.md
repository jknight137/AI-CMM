# Pull Request Checklist (Agent-Aware)

## Instructions

Copy this checklist into your PR or merge request template (e.g., `.github/PULL_REQUEST_TEMPLATE.md`, GitLab MR description template, or equivalent). All items apply to every PR. Items marked **(Agent)** are additional checks required when the PR includes agent-generated or agent-assisted code.

---

## PR Description

- [ ] PR title is clear and follows the convention: `type: short description` (e.g., `feat: add user export endpoint`)
- [ ] PR description explains **what** changed and **why**
- [ ] Related issue or ticket is linked (e.g., `Closes #123`)

## Agent Disclosure

- [ ] **Is this PR agent-generated or agent-assisted?** (Yes / No)
- If Yes:
  - [ ] **(Agent)** Agent tool used is identified (e.g., Claude Code, Copilot, Codex)
  - [ ] **(Agent)** Commits include a `Co-Authored-By` trailer or equivalent attribution
  - [ ] **(Agent)** PR description includes what the agent was asked to do (the task or prompt summary)
  - [ ] **(Agent)** PR description notes any limitations, assumptions, or areas where the agent output was modified by the developer

## Code Quality

- [ ] Code follows project conventions (style guide, naming, patterns)
- [ ] No unnecessary changes outside the scope of this PR
- [ ] No commented-out code left behind
- [ ] No TODO comments without a linked issue
- [ ] **(Agent)** Reviewed for over-abstraction (agents tend to create unnecessary helpers, wrappers, or interfaces)
- [ ] **(Agent)** Reviewed for hallucinated imports or dependencies (packages that do not exist or are not in the approved list)
- [ ] **(Agent)** Reviewed for incorrect error handling (agents sometimes swallow errors or use overly broad catch blocks)
- [ ] **(Agent)** Reviewed for copy-paste patterns that should use existing shared utilities

## Testing

- [ ] New functionality has corresponding tests
- [ ] Bug fixes have regression tests
- [ ] All tests pass locally
- [ ] Test coverage does not decrease
- [ ] **(Agent)** Agent-generated tests actually assert meaningful behavior (not just "does not throw")
- [ ] **(Agent)** Agent-generated tests do not duplicate existing test coverage

## Security

- [ ] No secrets, credentials, or API keys in the code
- [ ] No PII logged or exposed
- [ ] Input validation is present at trust boundaries
- [ ] Dependencies added are from approved sources and scanned for vulnerabilities
- [ ] **(Agent)** SAST scan passes with no new findings
- [ ] **(Agent)** No new permissions or IAM/RBAC changes without security team review
- [ ] **(Agent)** No changes to authentication or authorization logic without security team review

## Infrastructure as Code (if applicable)

- [ ] IaC validation passes (e.g., `terraform validate`)
- [ ] Linter passes with no new findings (e.g., tflint, tfsec, checkov)
- [ ] Plan output reviewed and posted to PR
- [ ] No wildcard IAM/RBAC actions or resources
- [ ] All resources have required tags/labels (Name, Environment, Team, ManagedBy)
- [ ] Regulatory compliance controls are met (if targeting a regulated environment)
- [ ] **(Agent)** Uses internal module registry modules where available (not raw resources)
- [ ] **(Agent)** Cost estimation reviewed (if Infracost or equivalent is configured)

## CI/CD (if modifying pipelines)

- [ ] Pipeline changes have been tested (e.g., on a feature branch with manual trigger)
- [ ] No new secrets or credentials added without platform team review
- [ ] No use of security-sensitive triggers or configurations without security review
- [ ] **(Agent)** Pipeline changes use shared/reusable components where available

## Documentation

- [ ] README updated if public API or configuration changed
- [ ] ADR created if an architectural decision was made
- [ ] Agent instruction file updated if project conventions changed
- [ ] **(Agent)** Agent-generated documentation reviewed for accuracy (agents may hallucinate features or API details)

## High-Risk Change Review (if applicable)

If this PR touches any high-risk category, the additional reviewers listed must approve:

| Category | Required Reviewer(s) |
|----------|---------------------|
| Identity/access policy changes | Security team + infrastructure team |
| Network security changes (firewalls, ACLs, security groups) | Security team |
| Database schema migrations | DBA/data team + application team |
| CI/CD pipeline modifications | Platform team |
| Secrets or credential handling | Security team |
| Authentication/authorization logic | Security team + application team |
| Production infrastructure (IaC) | Infrastructure team + change advisory |
| Regulated environment changes | Compliance team + security team |
| New dependency additions / major version upgrades | Application team lead |
| Data pipeline or ETL changes | Data team + compliance |

- [ ] High-risk categories identified (list them): _______________
- [ ] Required additional reviewers added to this PR

## Final Reviewer Checklist

- [ ] I have read the full diff (not just the PR description)
- [ ] I understand what the code does and why
- [ ] I have verified the tests are meaningful
- [ ] I am satisfied this PR meets our quality bar
- [ ] **(Agent)** I have paid extra attention to areas where agents commonly make mistakes (see above)
