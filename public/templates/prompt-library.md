# Prompt Library for AI Coding Agents

## How to Use This Library

These prompts are designed to be used with AI coding agents (Claude Code, Copilot, Codex, Cursor) for common development tasks. Each prompt includes:

- **Context needed**: what the agent needs access to before you run the prompt.
- **The prompt**: copy-paste or adapt to your task.
- **What to verify**: what a human should check in the output.

Customize these prompts for your specific repo by adding them to your CLAUDE.md or prompt snippets file.

---

## Category 1: Code Refactoring

### Prompt 1.1: Extract Function

**Context needed**: The file containing the code to refactor. CLAUDE.md with project conventions.

**Prompt**:
```
Read [FILE_PATH]. The block of code between lines [START] and [END] should be extracted
into a separate function. Follow the naming conventions in CLAUDE.md. The new function
should:
- Have a clear, descriptive name
- Accept only the parameters it needs (no passing large objects when only one field is used)
- Include type annotations consistent with the rest of the file
- Have a unit test in the corresponding test file

Do not change any behavior. This is a pure refactor.
```

**What to verify**: Behavior is unchanged. Tests pass. No new dependencies introduced.

### Prompt 1.2: Reduce Complexity

**Context needed**: The file with the complex function. Linter output showing complexity score.

**Prompt**:
```
Read [FILE_PATH]. The function [FUNCTION_NAME] has a cyclomatic complexity of [SCORE].
Refactor it to reduce complexity below [TARGET_SCORE]. Allowed techniques:
- Early returns for guard clauses
- Extract helper functions (private, in the same module)
- Replace nested conditionals with lookup tables or pattern matching
- Simplify boolean logic

Do not change external behavior. Do not add new public API surface. Keep all existing tests passing.
```

**What to verify**: Complexity score decreased. Tests pass. No behavior change.

---

## Category 2: Test Generation

### Prompt 2.1: Add Unit Tests for Uncovered Code

**Context needed**: The source file. The existing test file (if any). Coverage report showing uncovered lines.

**Prompt**:
```
Read [SOURCE_FILE] and [TEST_FILE]. The following lines are not covered by tests:
[LINE_NUMBERS].

Write unit tests to cover these lines. Follow the existing test patterns in [TEST_FILE]:
- Use the same test framework and assertion style
- Use the same fixture/setup patterns
- Name tests descriptively: test_[function]_[scenario]_[expected_outcome]

Include edge cases:
- Null/empty input
- Boundary values
- Error conditions

Do not modify the source file. Only add tests.
```

**What to verify**: New tests pass. Coverage improves on the target lines. Tests are not trivially passing (actually assert meaningful behavior).

### Prompt 2.2: Generate Integration Test

**Context needed**: The service or module to test. API contracts or interface definitions. Test infrastructure documentation.

**Prompt**:
```
Read [SERVICE_FILE] and [API_SPEC_OR_INTERFACE]. Write an integration test that verifies:
- The happy path for [ENDPOINT_OR_FUNCTION]
- At least one error path (invalid input, downstream failure)
- Response structure matches the contract

Use the test infrastructure described in [TEST_INFRA_DOC]. Follow these constraints:
- Tests must be idempotent (clean up after themselves)
- Use test fixtures, not production data
- Mock external dependencies that are outside the test boundary
- Tests must run in CI within [TIMEOUT] seconds
```

**What to verify**: Test passes locally and in CI. Test is actually testing the integration, not just mocking everything. Cleanup works.

---

## Category 3: Fix CI

### Prompt 3.1: Diagnose CI Failure

**Context needed**: CI log output (paste relevant section). The workflow file. The failing step.

**Prompt**:
```
Here is a CI failure log from GitHub Actions:

[PASTE CI LOG]

The failing step is [STEP_NAME] in workflow [WORKFLOW_FILE].

Diagnose the root cause. Consider:
- Dependency version changes
- Environment differences (CI vs. local)
- Flaky test patterns
- Missing environment variables or secrets
- Permission issues

Propose a fix. If the fix involves changing the workflow file, show the diff.
If the fix involves changing application code, show the diff and explain why.
Do not propose suppressing the error without fixing the root cause.
```

**What to verify**: The diagnosis matches the actual failure. The fix addresses root cause, not symptoms.

### Prompt 3.2: Optimize Pipeline Speed

**Context needed**: The workflow file. Pipeline timing data (which steps are slowest).

**Prompt**:
```
Read [WORKFLOW_FILE]. Here are the step durations from the last 5 runs:

[PASTE TIMING DATA]

Propose optimizations to reduce total pipeline time. Consider:
- Caching (dependencies, build artifacts, Docker layers)
- Parallelizing independent steps
- Reducing unnecessary steps
- Using smaller/faster runners for simple steps
- Conditional execution (skip steps when irrelevant files changed)

For each optimization, estimate the time saved and note any risks.
Do not remove any security-related steps (secrets scan, SAST, dependency audit).
```

**What to verify**: Optimizations are safe. No security steps removed. Estimated savings are reasonable.

---

## Category 4: Terraform Module

### Prompt 4.1: Generate Terraform Module

**Context needed**: CLAUDE.md with IaC conventions. Existing module examples. AWS region and account constraints.

**Prompt**:
```
Create a Terraform module for [RESOURCE_DESCRIPTION]. Follow the conventions in CLAUDE.md
and use the structure from [EXAMPLE_MODULE_PATH] as a reference.

Requirements:
- Module structure: main.tf, variables.tf, outputs.tf, README.md
- Use modules from the internal registry ([REGISTRY_URL]) where available
- All resources must have tags: Name, Environment, Team, ManagedBy=terraform
- No wildcard IAM actions or resources
- Encryption at rest and in transit where applicable
- Logging enabled where applicable

For Federal environments, also include:
- [LIST_FEDERAL_SPECIFIC_REQUIREMENTS, e.g., KMS CMK encryption, VPC flow logs, access logging]

Output the complete module files. Include variable descriptions and sensible defaults.
```

**What to verify**: Module passes `terraform validate` and tflint/tfsec. No wildcard IAM. Tags are correct. Federal requirements are met. Module uses internal registry modules where appropriate.

### Prompt 4.2: Review Terraform Plan

**Context needed**: `terraform plan` output. The module source code.

**Prompt**:
```
Review this Terraform plan output:

[PASTE PLAN OUTPUT]

Check for:
- Resources being destroyed unexpectedly (should be zero unless this is intentional)
- IAM policy changes (flag any permission escalation)
- Security group changes (flag any new ingress from 0.0.0.0/0)
- Cost implications (new resources, size changes)
- Missing tags on new resources
- Changes that affect availability (instance type changes, AZ changes)

Summarize:
1. Safe changes (no risk)
2. Changes that need attention (review carefully)
3. Dangerous changes (should block until reviewed)
```

**What to verify**: Agent correctly identified all resource changes. Dangerous changes are actually dangerous. Cost implications are reasonable.

---

## Category 5: Incident Triage

### Prompt 5.1: Log Analysis

**Context needed**: CloudWatch log excerpt. Service architecture description. Recent deployment history.

**Prompt**:
```
Analyze these logs from [SERVICE_NAME]:

[PASTE LOG EXCERPT]

Context:
- Service architecture: [BRIEF_DESCRIPTION]
- Last deployment: [TIMESTAMP]
- Current symptoms: [DESCRIBE_SYMPTOMS]

Provide:
1. Timeline of events based on log evidence
2. Likely root cause (with confidence level: high/medium/low)
3. Suggested next diagnostic steps (what logs or metrics to check next)
4. If root cause is clear, suggest a remediation approach

Do not suggest any remediation that requires production write access.
All suggestions should be things the on-call engineer can verify and execute.
```

**What to verify**: Timeline is accurate. Root cause hypothesis is evidence-based. Remediation suggestions are safe and actionable.

### Prompt 5.2: Draft Postmortem

**Context needed**: Incident timeline. Resolution steps. Impact data. Monitoring alerts.

**Prompt**:
```
Draft a postmortem for the following incident:

Incident ID: [ID]
Duration: [START] to [END]
Impact: [DESCRIBE_IMPACT]
Timeline: [PASTE_TIMELINE]
Resolution: [DESCRIBE_RESOLUTION]

Use this template structure:
1. Summary (2-3 sentences)
2. Impact (users affected, duration, severity)
3. Timeline (chronological, with timestamps)
4. Root cause (technical explanation)
5. Resolution (what was done to fix it)
6. Contributing factors (what made the incident worse or delayed resolution)
7. Action items (specific, assigned, with due dates marked as TBD)
8. Lessons learned

Be factual. Do not speculate beyond the evidence provided. Mark any assumptions clearly.
Flag if any information seems missing from the timeline.
```

**What to verify**: Facts match the actual incident. No hallucinated details. Action items are actionable (not vague). Missing information is flagged, not invented.

---

## Category 6: Documentation

### Prompt 6.1: Update README After Code Changes

**Context needed**: The changed files (diff). The current README.

**Prompt**:
```
Read the current README at [README_PATH] and the following code changes:

[PASTE DIFF OR LIST CHANGED FILES]

Update the README to reflect these changes. Specifically:
- Update any usage examples that reference changed APIs
- Update configuration documentation if config options changed
- Update architecture description if structure changed
- Do not add sections that did not exist before unless the changes clearly require them
- Keep the existing tone and style

Show only the sections that changed, with enough context to locate them in the file.
```

**What to verify**: Changes are accurate. No hallucinated features. Style matches existing README.

### Prompt 6.2: Write ADR

**Context needed**: The decision being documented. Alternatives considered. Context and constraints.

**Prompt**:
```
Write an Architecture Decision Record (ADR) for the following decision:

Decision: [DESCRIBE_DECISION]
Context: [WHY_THIS_DECISION_WAS_NEEDED]
Alternatives considered: [LIST_ALTERNATIVES]
Constraints: [LIST_CONSTRAINTS]
Chosen approach: [WHICH_ALTERNATIVE_AND_WHY]

Use this ADR format:
# ADR-[NUMBER]: [TITLE]
## Status: [Proposed/Accepted/Deprecated/Superseded]
## Context
## Decision
## Alternatives Considered
## Consequences
## References

Be specific about trade-offs. State what we are giving up, not just what we are gaining.
```

**What to verify**: Decision rationale is accurate. Trade-offs are honest. Alternatives are fairly represented.

---

## Category 7: Security

### Prompt 7.1: Security Review of Code Change

**Context needed**: The diff or changed files. OWASP top 10 reference. Project security requirements.

**Prompt**:
```
Review this code change for security issues:

[PASTE DIFF OR FILE]

Check for:
- Injection vulnerabilities (SQL, command, LDAP, XSS)
- Authentication/authorization flaws
- Sensitive data exposure (logging PII, hardcoded secrets, unencrypted storage)
- Insecure deserialization
- Missing input validation at trust boundaries
- Dependency vulnerabilities (if new dependencies added)
- Misconfigured security headers (if web-facing)

For each finding:
- Severity: Critical / High / Medium / Low
- Location: file and line
- Description: what the issue is
- Recommendation: how to fix it

If no issues found, state that explicitly. Do not invent findings.
```

**What to verify**: Findings are real (not false positives). Severity ratings are appropriate. Recommendations are correct.
