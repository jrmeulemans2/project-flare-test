# Implementation Plan: Project Flare Static Landing Page

**Branch**: `001-project-flare-landing` | **Date**: 2025-03-02 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `specs/001-project-flare-landing/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Deliver a low-cost, highly secure static landing page for Project Flare. Static content is hosted in an **Azure Storage Account** (static website) and served via **Azure Front Door** (East US, Terraform). CI enforces Constitution compliance on PRs: every spec folder must have **solution.yaml** with **metadata.region = eastus**, validated by a GitHub Action using a YAML linter and region check.

## High-Level Architecture

```text
Internet → Azure Front Door (East US) → Azure Storage Account – Static Website (East US)
```

**Flow**: User request → Front Door (HTTPS, optional WAF/caching) → Storage $web origin → static files. No compute.

## Technical Context

**Language/Version**: HTML5, CSS3 (static assets; no runtime language required)  
**Primary Dependencies**: Azure Storage Account (static website), Azure Front Door  
**Storage**: Azure Blob Storage ($web container for static website)  
**Testing**: Manual browser verification; optional static lint (HTML/CSS); CI validates solution.yaml (yamllint + region=eastus)  
**Target Platform**: Web browsers; Azure East US  
**Project Type**: static-website  
**Performance Goals**: First content within 5 seconds on typical broadband (per spec SC-001)  
**Constraints**: East US only; Terraform-only IaC; static-only; no personal data collection  
**Scale/Scope**: Single landing page; low traffic; high security and low cost

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|--------|
| I. Azure-Only | Pass | Storage Account and Front Door are Azure-only. |
| II. East US Region | Pass | All resources use `location = "eastus"`; CI enforces region in solution.yaml. |
| III. Terraform IaC | Pass | All infra defined in Terraform; no manual production resources. |
| IV. Static-First | Pass | Content served from Storage static website; no server-side execution. |
| V. Simplicity & Traceability | Pass | Two main Azure services; solution.yaml and CI document compliance. |

No violations.

### CI: Solution YAML compliance (PR)

A **GitHub Action** runs on every **pull request** to enforce Constitution II (East US) and spec-folder consistency:

- **Workflow**: `.github/workflows/solution-yaml-compliance.yml`
- **Trigger**: `pull_request` to `main` or `master`
- **Checks**:
  1. **Existence**: Every folder under `specs/*/` MUST contain a `solution.yaml` file.
  2. **YAML Linter**: Each `solution.yaml` is linted with **yamllint** (syntax and style).
  3. **Region**: Each file MUST have `metadata.region` equal to **eastus** (Constitution II).

Failures are reported with `::error` annotations; the PR cannot merge until the workflow passes.

## Project Structure

### Documentation (this feature)

```text
specs/001-project-flare-landing/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── solution.yaml
├── infrastructure.yaml
├── contracts/
└── tasks.md
```

### Source Code (repository root)

```text
.github/
└── workflows/
    └── solution-yaml-compliance.yml   # PR check: solution.yaml exists, yamllint, region=eastus

site/
├── index.html
├── css/
│   └── styles.css
└── assets/

terraform/
├── main.tf
├── variables.tf
├── outputs.tf
└── (backend config for state in Azure Storage; see quickstart)
```

**Structure Decision**: Static site in `site/`, Terraform in `terraform/`, CI in `.github/workflows/`. No backend or tests directory required for static-only delivery.

## Complexity Tracking

Not applicable; no violations.
