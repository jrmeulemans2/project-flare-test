# Research: Project Flare Static Landing Page

**Feature**: 001-project-flare-landing  
**Date**: 2025-03-02

## 1. Hosting: Azure Storage Account Static Website

**Decision**: Use Azure Storage Account with static website hosting enabled ($web container) as the origin for the landing page.

**Rationale**: Matches constitution (Azure-only, static-first, East US). Low cost (storage + egress only), no compute, minimal attack surface. Single HTML/CSS/assets in $web; no server-side execution. Supports index document and error document for SPA-like default route.

**Alternatives considered**:
- **Azure Static Web Apps**: More features (CI/CD, serverless APIs); higher complexity and cost for a single static page. Rejected for simplicity.
- **App Service Static Web**: Overkill and higher cost for static-only content. Rejected.
- **Blob-only (no static website)**: Would require custom routing; static website feature provides default document and 404 behavior out of the box. Rejected.

---

## 2. Delivery: Azure Front Door

**Decision**: Use Azure Front Door in front of the Storage static website origin.

**Rationale**: Single public endpoint with HTTPS (TLS termination), optional WAF, caching at the edge, and ability to hide the Storage endpoint (origin can be private). Aligns with spec FR-003 (secure channel) and SC-002 (all content over secure connection). East US placement per constitution.

**Alternatives considered**:
- **Storage static website URL only**: Simpler and cheaper but exposes the *.z6.web.core.windows.net URL and may have weaker default security posture. Front Door gives custom/clean URL and centralized HTTPS/WAF. Chosen for “highly secure” requirement.
- **Azure CDN (Standard Microsoft)**: Similar benefits; Front Door provides unified product for routing, WAF, and Azure integration. Front Door chosen for consistency and single control plane.
- **No CDN**: Higher latency and no edge caching. Rejected for performance (SC-001) and resilience.

---

## 3. Region and IaC

**Decision**: All resources in **East US** (`eastus`). All infrastructure in **Terraform** (AzureRM provider); state in Azure Storage with locking (e.g., blob container + state file).

**Rationale**: Constitution Principles II (East US) and III (Terraform IaC). No exceptions needed.

**Alternatives considered**: None; constitution is binding.

---

## 4. Security and Best Practices

**Decision**: (a) HTTPS only via Front Door; (b) Storage $web with least privilege (only Front Door or restricted origin access); (c) no personal data collection; (d) optional security headers via Front Door or minimal meta/code in static page.

**Rationale**: Spec FR-003, FR-004, FR-006 and SC-004. Static-first avoids server-side data handling; Front Door handles TLS and can enforce HTTPS redirect.

**Alternatives considered**: WAF rules (optional); add later if threat model requires. Not required for initial “highly secure” static page.
