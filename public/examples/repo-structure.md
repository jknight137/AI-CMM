# Recommended Repository Structure for AI Agent Integration

## Overview

This document describes the recommended files and directories that should be present in repositories to support AI agent usage effectively. It covers both application repositories and infrastructure (IaC) repositories, with examples for multiple SCM platforms.

---

## Application Repository Structure

```
my-service/
|
|-- .github/                                 # GitHub-specific (see platform alternatives below)
|   |-- PULL_REQUEST_TEMPLATE.md             # PR template with agent disclosure
|   |-- CODEOWNERS                           # Protects high-risk paths; requires reviews
|   |-- workflows/
|   |   |-- ci.yml                           # Standard CI pipeline (build, test, lint, SAST)
|   |   |-- agent-gates.yml                  # Agent-specific gate steps (see examples/ci-gates/)
|   |   |-- deploy.yml                       # Deployment pipeline (no agent involvement)
|   |
|-- .devcontainer/                           # Dev environment config (devcontainers)
|   |-- devcontainer.json                    # Config with agent tooling
|   |-- Dockerfile                           # Based on registry-hosted base image
|   |
|-- .claude/                                 # Claude Code project-level settings (optional)
|   |-- settings.json                        # Model preferences, permissions, etc.
|   |
|-- CLAUDE.md                               # Agent instructions for this project (see below)
|-- .gitleaks.toml                           # Secrets scanning configuration
|-- .tfsec.yml                               # IaC security scanner config (if IaC in this repo)
|--
|-- docs/
|   |-- adr/                                 # Architecture Decision Records
|   |   |-- 001-initial-architecture.md
|   |   |-- 002-database-selection.md
|   |   |-- ...
|   |-- architecture.md                      # Architecture overview (agent context)
|   |-- conventions.md                       # Coding conventions (agent context)
|   |-- api.md                               # API documentation (agent context)
|   |
|-- src/                                     # Application source code
|-- tests/                                   # Test files
|-- README.md                                # Project readme
|-- package.json / requirements.txt / go.mod # Dependency manifest
```

### Platform Alternatives for SCM-Specific Directories

| SCM Platform | CI Config Location | PR/MR Template | Code Ownership |
|--------------|-------------------|----------------|----------------|
| GitHub | `.github/workflows/*.yml` | `.github/PULL_REQUEST_TEMPLATE.md` | `.github/CODEOWNERS` |
| GitLab | `.gitlab-ci.yml` | `.gitlab/merge_request_templates/*.md` | `CODEOWNERS` (root or `docs/`) |
| Bitbucket | `bitbucket-pipelines.yml` | PR template in repo settings | N/A (use branch permissions) |
| Azure DevOps | `azure-pipelines.yml` | PR template in repo settings or `.azuredevops/pull_request_template.md` | Required reviewers via branch policy |

---

## Infrastructure (IaC) Repository Structure

```
infra/
|
|-- .github/                                 # (or equivalent for your SCM platform)
|   |-- PULL_REQUEST_TEMPLATE.md
|   |-- CODEOWNERS                           # Protects all .tf files; requires infra team review
|   |-- workflows/
|   |   |-- terraform-validate.yml           # Validate + lint on PR
|   |   |-- terraform-plan.yml               # Plan on PR (posts plan output as comment)
|   |   |-- terraform-apply.yml              # Apply on merge to main (with approval gate)
|   |   |-- agent-gates.yml                  # Agent-specific gate steps
|   |
|-- .devcontainer/
|   |-- devcontainer.json                    # Dev environment with IaC tools and agent tooling
|   |-- Dockerfile
|   |
|-- CLAUDE.md                               # Agent instructions for IaC work (see below)
|--
|-- modules/                                 # Reusable IaC modules
|   |-- networking/
|   |   |-- main.tf
|   |   |-- variables.tf
|   |   |-- outputs.tf
|   |   |-- README.md
|   |-- compute/
|   |-- storage/
|   |-- iam/                                 # IAM modules (high-risk, ownership-protected)
|   |-- ...
|
|-- environments/
|   |-- standard/                            # Standard (non-regulated) environments
|   |   |-- dev/
|   |   |   |-- main.tf
|   |   |   |-- terraform.tfvars
|   |   |   |-- backend.tf
|   |   |-- staging/
|   |   |-- production/
|   |
|   |-- regulated/                           # Separate directory tree for regulated environments
|   |   |-- dev/
|   |   |   |-- main.tf
|   |   |   |-- terraform.tfvars             # Regulated-environment-specific variables
|   |   |   |-- backend.tf                   # Backend in approved region/boundary
|   |   |-- staging/
|   |   |-- production/
|
|-- policies/                                # OPA/Sentinel policies (Level 4+)
|   |-- deny-wildcard-iam.rego
|   |-- require-tags.rego
|   |-- require-encryption.rego
|   |-- regulated-controls.rego
|
|-- docs/
|   |-- adr/
|   |-- module-guide.md                      # How to use and create modules
|   |-- conventions.md                       # IaC conventions and naming
```

---

## Key Files Explained

### CLAUDE.md (Application Repo Example)

This is the primary file that gives AI agents context about your project. It should be specific, not generic. Other agents may use different file names (e.g., `.github/copilot-instructions.md`), but the content principles are the same.

```markdown
# Project: my-service

## What This Is
A REST API service that handles [DESCRIPTION]. Runs on [COMPUTE_PLATFORM, e.g.,
Kubernetes, ECS, Cloud Run, serverless]. Written in [LANGUAGE/FRAMEWORK].

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
- Only add dependencies from the approved list in the artifact registry.
- Do not use `--force` or `--legacy-peer-deps` to resolve conflicts. Fix them properly.

## What NOT to Do
- Do not modify CI/CD pipeline files (protected by code ownership rules).
- Do not add IAM/RBAC policies or security rules. Those go in the infra repo.
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
IaC (Terraform) infrastructure for the organization. Manages cloud resources for
both standard and regulated environments.

## Structure
- modules/: Reusable modules. Use these instead of writing raw resources.
- environments/standard/: Standard environment configurations.
- environments/regulated/: Regulated environment configurations (stricter controls).
- policies/: OPA policies enforced in CI.

## Conventions
- Resource naming: [project]-[environment]-[resource_type]-[purpose]
- All resources must have tags/labels: Name, Environment, Team, ManagedBy=terraform
- Use variables for everything that differs between environments.
- Use locals for computed values that are derived from variables.
- Module README.md must include: purpose, inputs, outputs, example usage.

## Security Rules
- No wildcard IAM/RBAC actions ("Action": "*") or resources ("Resource": "*").
- No inline IAM policies. Use managed policy resources.
- All storage buckets: encryption enabled, public access blocked.
- All database instances: encryption at rest, no public accessibility.
- All security groups/firewall rules: no ingress from 0.0.0.0/0 unless explicitly
  approved in an ADR.

## Regulated Environment Rules
- Regulated environments use the approved provider configuration and region.
- Encryption keys must use service-managed or customer-managed keys (not default encryption).
- Storage buckets must have access logging enabled.
- VPCs/networks must have flow logs enabled.
- All regulated environment changes require compliance team review.

## What NOT to Do
- Do not create raw resources when a module exists in modules/.
- Do not hardcode account IDs, region names, or resource ARNs. Use variables or data sources.
- Do not modify backend.tf files (managed by platform team).
- Do not create cross-environment dependencies (standard resources must not reference
  regulated resources or vice versa).
```

### Code Ownership Example (CODEOWNERS Format)

This example uses the CODEOWNERS format supported by GitHub and GitLab. Adapt for your platform.

```
# Default: require one reviewer from the team
*                           @org/my-team

# CI/CD pipelines: require platform team review
.github/workflows/          @org/platform-team
.gitlab-ci.yml              @org/platform-team

# IAM and security: require security team review
**/iam/                     @org/security-team @org/infra-team
**/iam*.tf                  @org/security-team @org/infra-team
*security_group*            @org/security-team
*firewall*                  @org/security-team

# Regulated environments: require compliance team review
environments/regulated/     @org/compliance-team @org/security-team @org/infra-team

# Database migrations: require DBA review
**/migrations/              @org/data-team @org/my-team
```

### devcontainer.json Example

This example uses the devcontainer spec, which is supported by VS Code, GitHub Codespaces, Gitpod (with adapter), and JetBrains Gateway.

```json
{
  "name": "my-service-dev",
  "image": "your-registry.example.com/devcontainers/base:latest",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
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
    "ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}"
  },
  "mounts": [],
  "remoteUser": "vscode"
}
```

**Note**: The `ANTHROPIC_API_KEY` should be injected from your secrets management service during developer workstation setup. Developers should not set this manually or use personal API keys.

### Alternative Dev Environment Approaches

| Approach | How Agent Tooling Is Configured | Notes |
|----------|-------------------------------|-------|
| **Devcontainers** (VS Code, Codespaces) | `devcontainer.json` with extensions and features | Best for reproducibility. Config is version-controlled. |
| **Gitpod** | `.gitpod.yml` with tasks and extensions | Cloud-native. Auto-provisions. |
| **JetBrains Gateway** | Remote dev with plugin configuration | IDE-specific. Use `.idea/` project settings. |
| **Nix / devbox** | `devbox.json` or `flake.nix` with tool packages | Language-agnostic. Reproducible. |
| **Manual setup** | README with setup instructions | Least reproducible. Use only if containerized options are not feasible. |

---

## Checklist: Is Your Repo Agent-Ready?

Use this checklist when onboarding a repository for AI agent usage.

- [ ] Agent instruction file (CLAUDE.md or equivalent) exists and is specific to this project (not a generic template)
- [ ] PR/MR template includes agent disclosure field
- [ ] Code ownership rules protect high-risk paths (pipelines, IAM/RBAC, auth, regulated environments)
- [ ] Dev environment is configured with agent tooling (devcontainer, Gitpod, or equivalent)
- [ ] Secrets scanning config (`.gitleaks.toml` or equivalent) is present
- [ ] Agent gate steps are present in CI (or equivalent checks in main CI pipeline)
- [ ] `docs/adr/` directory exists with at least one ADR
- [ ] `docs/` contains architecture and conventions documentation referenced by agent instructions
- [ ] For IaC repos: linter configs (tflint, tfsec/checkov) are present
- [ ] For IaC repos: regulated and standard environments are clearly separated
- [ ] Branch/merge protection requires at least one approval and passing CI checks
