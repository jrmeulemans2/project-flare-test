# Implementation Plan: Project Flare Static Landing Page

**Branch**: `001-project-flare-landing` | **Date**: 2026-03-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-project-flare-landing/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Deliver a single static landing page for Project Flare that is low-cost and highly secure. Use Azure Storage Account static website ($web) as origin and Azure Front Door for HTTPS, caching, and WAF. All infrastructure in East US via Terraform. Enterprise-ready security: (1) HTTPS enforcement and minimum TLS 1.2, (2) WAF policy on Front Door, (3) Managed Identity and Storage restricted to Front Door only. Observability: Log Analytics Workspace and Diagnostic Settings for Front Door and Storage. See [security.md](./security.md) and [infrastructure.yaml](./infrastructure.yaml).

## Technical Context

**Language/Version**: HTML5, CSS3 (static assets; no runtime language required)  
**Primary Dependencies**: None (static files only)  
**Storage**: Azure Storage Account $web container (static website); no database  
**Testing**: Manual acceptance (spec scenarios); optional static validation / link checks  
**Target Platform**: Web browsers; delivery via Azure Front Door (edge)  
**Project Type**: Static website (landing page)  
**Performance Goals**: First content visible &lt;5s (SC-001); 99% uptime (SC-003)  
**Constraints**: Encryption in transit only (FR-003); no server-side execution or personal data (FR-004, SC-004)  
**Scale/Scope**: Single landing page; public anonymous read; minimal scale  
**Observability**: Log Analytics Workspace; Diagnostic Settings for Front Door and Storage ([spec.md § Diagnostic Settings](./spec.md), [research.md §6](./research.md#6-log-analytics-workspace-and-diagnostic-settings)).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|--------|
| **I. Azure-Only** | Pass | Storage, Front Door, WAF, Log Analytics, diagnostics—all Azure. |
| **II. East US** | Pass | All resources `location: eastus` (Storage, Front Door profile, WAF policy, LAW). |
| **III. Terraform IaC** | Pass | All infra defined for Terraform; no manual production changes. |
| **IV. Static-First** | Pass | Static HTML/CSS in $web; no server-side execution. |
| **V. Simplicity & Traceability** | Pass | Security and observability documented in [security.md](./security.md), [research.md](./research.md), [spec.md](./spec.md). |

No exceptions. Re-check after Phase 1: unchanged.

## Infrastructure & Constitution Review

*Review of [infrastructure.yaml](./infrastructure.yaml): security enhancements, observability enhancements, and Day 1 Constitution alignment.*

### 1. Do the 3 security enhancements appear in infrastructure.yaml?

| Enhancement | In infrastructure.yaml? | Where |
|------------|-------------------------|--------|
| **1. HTTPS enforcement + minimum TLS 1.2** | Yes | `cdn-flare-prod`: comments "HTTPS only, minimum TLS 1.2" and Terraform notes (`accepted_protocols ["Https"]`, `minimum_tls_version 1.2`). Bottom `security` list: `https_enforcement_and_minimum_tls_1_2`. |
| **2. WAF policy on Front Door** | Yes | `waf-flare-prod` (firewall policy, mode Prevention), `waf-flare-prod-security-policy` (links WAF to profile). Comments reference security.md §2. Bottom list: `waf_policy_on_front_door`. |
| **3. Managed Identity + Storage restriction** | Yes | **Storage** `stflareprod001`: `allow_nested_items_to_be_public: false`, comment for network_rules (Front Door / trusted services only). **Front Door** `cdn-flare-prod`: comments for Managed Identity and origin managed_identity auth. Bottom list: `managed_identity_and_storage_restriction`. |

**Conclusion**: The updated infrastructure.yaml **does** include all three security enhancements (as resource definitions and/or comments and the `security` checklist).

### 2. Do the observability enhancements appear in infrastructure.yaml?

| Enhancement | In infrastructure.yaml? | Where |
|-------------|--------------------------|--------|
| **Log Analytics Workspace** | Yes | `law-flare-prod` (lines 28–31): `azurerm_log_analytics_workspace`, `location: eastus`, central log destination. |
| **Front Door diagnostic settings** | Yes | `diag-frontdoor-flare-prod` (32–35): target cdn-flare-prod, destination law-flare-prod; logs: FrontDoorAccessLog, FrontDoorHealthProbeLog, FrontDoorWebApplicationFirewallLog. |
| **Storage blob diagnostic settings** | Yes | `diag-storage-blob-flare-prod` (36–39): target stflareprod001/blobServices/default, destination law-flare-prod; logs: StorageRead, StorageWrite, StorageDelete; metrics: Transaction. |

**Conclusion**: The updated infrastructure.yaml **does** include the observability enhancements (Log Analytics Workspace and both Diagnostic Settings for Front Door traffic and Storage Account access).

### 3. Any conflicting settings with the Day 1 Constitution?

| Principle | Check | Result |
|-----------|--------|--------|
| **I. Azure-Only** | All resources are Azure (Storage, Front Door, WAF, LAW, diagnostic settings). | No conflict. |
| **II. East US** | Storage, Front Door profile, WAF policy, and Log Analytics Workspace all have `location: eastus`. Diagnostic settings inherit from their target resources. | No conflict. |
| **III. Terraform IaC** | infrastructure.yaml is the spec for Terraform; all resources are Terraform-managed. No manual-only resources. | No conflict. |
| **IV. Static-First** | Content remains static in $web; Front Door serves static assets. Storage firewall / MI do not introduce server-side execution. | No conflict. |
| **V. Simplicity & Traceability** | WAF, Managed Identity, and Storage restriction add some complexity but are justified for "Enterprise Ready" and are documented in security.md and research.md. | No conflict; additions are traceable. |

**Conclusion**: There are **no conflicting settings** between the new security and observability posture and the Day 1 Constitution. All principles (I–V) remain satisfied; the security and observability additions are documented and aligned with Platform & Compliance Constraints (Azure, East US, Terraform, static hosting).

## Project Structure

### Documentation (this feature)

```text
specs/001-project-flare-landing/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── security.md          # Security specification (Enterprise-ready configs)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
frontend/
├── site/                # Static landing page assets
│   ├── index.html
│   ├── css/
│   └── assets/
└── (tests as needed)

terraform/               # IaC (Terraform); backend config + modules/resources
```

**Structure Decision**: Static-only frontend under `frontend/site/`. Terraform in `terraform/` for Storage, Front Door, WAF, Log Analytics, and Diagnostic Settings. No backend or API; contracts describe landing page content only.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | — | — |
