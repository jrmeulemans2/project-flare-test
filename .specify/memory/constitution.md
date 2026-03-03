<!--
  Sync Impact Report
  Version change: (none) → 1.0.0
  Modified principles: N/A (initial creation)
  Added sections: Core Principles (5), Platform & Compliance Constraints, Development Workflow, Governance
  Removed sections: None
  Templates: plan-template.md ✅ (Constitution Check gate aligns); spec-template.md ✅ (no mandatory section changes); tasks-template.md ✅ (task types compatible; IaC/foundational tasks implied); checklist-template.md ✅ (no constitution refs); commands/*.md — none present
  Follow-up TODOs: None
-->

# Project Flare Constitution

## Core Principles

### I. Azure-Only

All infrastructure and hosting MUST use Microsoft Azure. No other cloud provider or bare-metal deployment is permitted.

**Rationale**: Ensures a single vendor strategy, consistent tooling, and unified billing and governance.

### II. East US Region (Compliance)

All Azure resources MUST be deployed in the **East US** region. No resources may be provisioned in other regions unless an explicit, documented exception is approved via the Governance amendment process.

**Rationale**: Compliance and data-residency requirements mandate East US; violations risk audit failure.

### III. Terraform IaC

All infrastructure MUST be defined and managed via Terraform. Production resources MUST NOT be created or modified manually in the Azure portal or CLI for ongoing use; Terraform is the single source of truth for infrastructure state.

**Rationale**: Reproducibility, drift prevention, and version-controlled change management require declarative IaC.

### IV. Static-First

Content MUST be served as static assets (e.g., static site hosting, CDN, blob storage) where possible. Server-side execution (e.g., Functions, App Service) is permitted only when justified and documented in the feature spec or plan.

**Rationale**: Static delivery reduces cost, complexity, and attack surface for a static website project.

### V. Simplicity & Traceability

Architecture MUST favor simple, minimal solutions. Any deviation from these principles or addition of complexity MUST be documented with rationale and, where applicable, a compliance or migration note.

**Rationale**: Keeps the project maintainable and ensures decisions are auditable for compliance reviews.

## Platform & Compliance Constraints

- **Cloud**: Azure only (see Principle I).
- **Region**: East US only (see Principle II). All Terraform providers and resource `location` (or equivalent) MUST use `East US` or `eastus`.
- **IaC**: Terraform only for provisioning and changes (see Principle III). State MUST be stored in a secure backend (e.g., Azure Storage with state locking).
- **Hosting**: Prefer Azure Static Web Apps, Azure Blob Storage + CDN, or equivalent static hosting; avoid dynamic compute unless justified.

## Development Workflow

- **Plans and specs**: Must include a Constitution Check that verifies Principles I–V (Azure, East US, Terraform, static-first, simplicity).
- **Infrastructure changes**: Must be made via Terraform changes only; PRs that add or modify Azure resources must include Terraform updates and pass plan/apply validation.
- **Compliance**: Any new dependency or region/tooling choice must be checked against this constitution; violations require amendment or documented exception.

## Governance

- This constitution supersedes ad-hoc practices for infrastructure, region, and tooling choices.
- **Amendments**: Require a documented proposal, rationale, and version bump per semantic versioning (MAJOR: backward-incompatible principle removal/change; MINOR: new principle or material expansion; PATCH: clarifications, typos).
- **Compliance**: All PRs and reviews MUST verify alignment with Principles I–V and the Platform & Compliance Constraints. Complexity or exceptions MUST be justified in the plan or spec.
- **Dates**: Ratification and last-amended dates are maintained in the version line below.

**Version**: 1.0.0 | **Ratified**: 2025-03-02 | **Last Amended**: 2025-03-02
