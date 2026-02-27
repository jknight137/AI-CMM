# Recommended Repository Structure for AI Agent Integration

## Overview

This document describes the recommended files and directories that should be present in repositories to support AI agent usage effectively. It covers both application repositories and infrastructure (IaC) repositories.

---

## Application Repository Structure

```
my-service/
|
|-- .github/
|   |-- PULL_REQUEST_TEMPLATE.md          # PR template with agent disclosure (see templates/pr-checklist.md)
|   |-- CODEOWNERS                         # Protects high-risk paths; requires reviews
|   |-- workflows/
|   |   |-- ci.yml                         # Standard CI pipeline (build, test, lint, SAST)
|   |   |-- agent-gates.yml                # Agent-specific gate steps (see examples/github-actions/)
|   |   |-- deploy.yml                     # Deployment pipeline (no agent involvement)
|   |
|-- .devcontainer/
|   |-- devcontainer.json                  # Devcontainer config with agent tooling
|   |-- Dockerfile                         # Based on Artifactory-hosted base image
|   |
|-- .claude/                               # Claude Code project-level settings (optional)
|   |-- settings.json                      # Model preferences, permissions, etc.
|   |
|-- CLAUDE.md                              # Agent instructions for this project (see below)
|-- .gitleaks.toml                         # Secrets scanning configuration
|-- .tfsec.yml                             # IaC security scanner config (if IaC in this repo)
|--
|-- docs/
|   |-- adr/                               # Architecture Decision Records
|   |   |-- 001-initial-architecture.md
|   |   |-- 002-database-selection.md
|   |   |-- ...
|   |-- architecture.md                    # Architecture overview (agent context)
|   |-- conventions.md                     # Coding conventions (agent context)
|   |-- api.md                             # API documentation (agent context)
|   |
|-- src/                                   # Application source code
|-- tests/                                 # Test files
|-- README.md                              # Project readme
|-- package.json / requirements.txt / go.mod  # Dependency manifest
```

---

## Infrastructure (IaC) Repository Structure

```
infra/
|
|-- .github/
|   |-- PULL_REQUEST_TEMPLATE.md
|   |-- CODEOWNERS                         # Protects all .tf files; requires infra team review
|   |-- workflows/
|   |   |-- terraform-validate.yml         # Validate + lint on PR
|   |   |-- terraform-plan.yml             # Plan on PR (posts plan output as comment)
|   |   |-- terraform-apply.yml            # Apply on merge to main (with approval gate)
|   |   |-- agent-gates.yml                # Agent-specific gate steps
|   |
|-- .devcontainer/
|   |-- devcontainer.json                  # Devcontainer with Terraform, tflint, tfsec, agent tools
|   |-- Dockerfile
|   |
|-- CLAUDE.md                              # Agent instructions for IaC work (see below)
|--
|-- modules/                               # Reusable Terraform modules
|   |-- networking/
|   |   |-- main.tf
|   |   |-- variables.tf
|   |   |-- outputs.tf
|   |   |-- README.md
|   |-- compute/
|   |-- storage/
|   |-- iam/                               # IAM modules (high-risk, CODEOWNERS-protected)
|   |-- ...
|
|-- environments/
|   |-- commercial/
|   |   |-- dev/
|   |   |   |-- main.tf
|   |   |   |-- terraform.tfvars
|   |   |   |-- backend.tf
|   |   |-- staging/
|   |   |-- production/
|   |
|   |-- federal/                           # Separate directory tree for Federal
|   |   |-- dev/
|   |   |   |-- main.tf
|   |   |   |-- terraform.tfvars           # Federal-specific variables
|   |   |   |-- backend.tf                 # GovCloud backend
|   |   |-- staging/
|   |   |-- production/
|
|-- policies/                              # OPA/Sentinel policies (Level 4+)
|   |-- deny-wildcard-iam.rego
|   |-- require-tags.rego
|   |-- require-encryption.rego
|   |-- federal-controls.rego
|
|-- docs/
|   |-- adr/
|   |-- module-guide.md                    # How to use and create modules
|   |-- conventions.md                     # Terraform conventions and naming
```

---

## Key Files Explained

### CLAUDE.md (Application Repo Example)

This is the primary file that gives AI agents context about your project. It should be specific, not generic.

```markdown
# Project: my-service

## What This Is
A REST API service that handles [DESCRIPTION]. Runs on ECS Fargate behind an ALB.
Written in [LANGUAGE/FRAMEWORK].

## Architecture
- Entry point: src/main.ts
- Routes: src/routes/
- Business logic: src/services/
- Data access: src/repositories/
- Tests: tests/ (mirrors src/ structure)

## Conventions
- Use camelCase for variables and functions, PascalCase for classes and types.
- All public functions must have JSDoc comments.
- Error handling: use AppError class from src/errors.ts. Do not throw raw Error objects.
- Database queries go in repository files, never in route handlers or services.
- All new endpoints need integration tests in tests/integration/.

## Testing
- Unit tests: `npm test`
- Integration tests: `npm run test:integration` (requires local Docker)
- Minimum coverage: 80% for new code.

## Dependencies
- Only add dependencies from the approved list in Artifactory.
- Do not use `--force` or `--legacy-peer-deps` to resolve conflicts. Fix them properly.

## What NOT to Do
- Do not modify .github/workflows/ files (CODEOWNERS-protected).
- Do not add IAM policies or security group rules. Those go in the infra repo.
- Do not log PII (names, emails, SSNs). Use the sanitizer from src/utils/sanitize.ts.
- Do not use wildcard imports.
- Do not create new top-level directories without discussing in an ADR.

## Related ADRs
- ADR-001: Initial architecture (docs/adr/001-initial-architecture.md)
- ADR-002: Database selection (docs/adr/002-database-selection.md)
```

### CLAUDE.md (IaC Repo Example)

```markdown
# Project: infra

## What This Is
Terraform infrastructure for [ORG_NAME]. Manages AWS resources for both commercial
and Federal environments.

## Structure
- modules/: Reusable modules. Use these instead of writing raw resources.
- environments/commercial/: Commercial AWS account environments.
- environments/federal/: GovCloud environments (stricter controls).
- policies/: OPA policies enforced in CI.

## Conventions
- Resource naming: [project]-[environment]-[resource_type]-[purpose]
- All resources must have tags: Name, Environment, Team, ManagedBy=terraform
- Use variables for everything that differs between environments.
- Use locals for computed values that are derived from variables.
- Module README.md must include: purpose, inputs, outputs, example usage.

## Security Rules
- No wildcard IAM actions ("Action": "*") or resources ("Resource": "*").
- No inline IAM policies. Use aws_iam_policy resources.
- All S3 buckets: encryption enabled (KMS for Federal, AES-256 minimum for commercial).
- All S3 buckets: public access blocked.
- All RDS instances: encryption at rest, no public accessibility.
- All security groups: no ingress from 0.0.0.0/0 unless explicitly approved in an ADR.

## Federal-Specific Rules
- Federal environments use GovCloud provider configuration (us-gov-west-1).
- Federal KMS keys must use AWS-managed or customer-managed CMKs (not default encryption).
- Federal S3 buckets must have access logging enabled.
- Federal VPCs must have flow logs enabled.
- All Federal changes require compliance team review.

## What NOT to Do
- Do not create raw resources when a module exists in modules/.
- Do not hardcode account IDs, region names, or ARNs. Use variables or data sources.
- Do not modify backend.tf files (managed by platform team).
- Do not create cross-environment dependencies (commercial resources must not reference Federal resources or vice versa).
```

### CODEOWNERS Example

```
# Default: require one reviewer from the team
*                           @org/my-team

# CI/CD workflows: require platform team review
.github/workflows/          @org/platform-team

# IAM and security: require security team review
**/iam/                     @org/security-team @org/infra-team
**/iam*.tf                  @org/security-team @org/infra-team
*security_group*            @org/security-team

# Federal environments: require compliance team review
environments/federal/       @org/compliance-team @org/security-team @org/infra-team

# Database migrations: require DBA review
**/migrations/              @org/data-team @org/my-team
```

### devcontainer.json Example

```json
{
  "name": "my-service-dev",
  "image": "artifactory.myorg.com/devcontainers/base:latest",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
    "ghcr.io/devcontainers/features/aws-cli:1": {},
    "ghcr.io/devcontainers/features/terraform:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "github.copilot",
        "anthropic.claude-code",
        "hashicorp.terraform",
        "dbaeumer.vscode-eslint"
      ],
      "settings": {
        "editor.formatOnSave": true
      }
    }
  },
  "postCreateCommand": "npm install",
  "containerEnv": {
    "ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}",
    "ARTIFACTORY_NPM_TOKEN": "${localEnv:ARTIFACTORY_NPM_TOKEN}"
  },
  "mounts": [],
  "remoteUser": "vscode"
}
```

**Note**: The `ANTHROPIC_API_KEY` is injected from the host environment, which retrieves it from AWS Secrets Manager during developer workstation setup. Developers do not set this manually.

---

## Checklist: Is Your Repo Agent-Ready?

Use this checklist when onboarding a repository for AI agent usage.

- [ ] CLAUDE.md exists and is specific to this project (not a generic template)
- [ ] .github/PULL_REQUEST_TEMPLATE.md includes agent disclosure field
- [ ] .github/CODEOWNERS protects high-risk paths (workflows, IAM, auth, Federal)
- [ ] .devcontainer/ is configured with agent tooling
- [ ] .gitleaks.toml (or equivalent secrets scanning config) is present
- [ ] agent-gates.yml workflow is present (or equivalent checks in main CI pipeline)
- [ ] docs/adr/ directory exists with at least one ADR
- [ ] docs/ contains architecture and conventions documentation referenced by CLAUDE.md
- [ ] For IaC repos: linter configs (tflint, tfsec/checkov) are present
- [ ] For IaC repos: Federal and commercial environments are clearly separated
- [ ] Branch protection requires at least one approval and passing CI checks
