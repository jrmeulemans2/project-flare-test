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
- **Storage static website URL only**: Simpler and cheaper but exposes the *.z6.web.core.windows.net URL and may have weaker default security posture. Front Door gives custom/clean URL and centralized HTTPS/WAF. Chosen for "highly secure" requirement.
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

**Alternatives considered**: WAF rules (optional); add later if threat model requires. Not required for initial "highly secure" static page.

---

## 5. Enterprise-Ready Security (3 Azure Configurations)

To make the project **Enterprise Ready**, three specific Azure security configurations are added to infrastructure and documented in [security.md](./security.md).

### 5.1 HTTPS Enforcement and Minimum TLS

**Decision**: Enforce HTTPS-only and minimum TLS 1.2 on Azure Front Door.

**Rationale**: Meets FR-003 and SC-002 (encryption in transit). Enterprise and compliance (e.g. PCI, SOC) expect no HTTP and no TLS 1.0/1.1. Front Door terminates TLS; configuring accepted protocols to HTTPS only and minimum TLS 1.2 satisfies this.

**Implementation (Terraform)**: On the Front Door profile / frontend endpoint (or route): set `accepted_protocols` to `["Https"]`, enable HTTPS and redirect HTTP→HTTPS where applicable, and set minimum TLS version to 1.2 in the TLS/custom domain policy.

**Alternatives considered**: HTTPS-only without TLS minimum (weaker); TLS 1.0/1.1 (deprecated, rejected).

---

### 5.2 WAF Policy on Front Door

**Decision**: Attach an Azure Front Door WAF (firewall) policy to the Front Door profile and apply it to the default route (e.g. `/*`).

**Rationale**: Enterprise readiness requires protection against common web attacks (OWASP Top 10), bots, and optional rate limiting. Front Door Standard supports `azurerm_cdn_frontdoor_firewall_policy` and `azurerm_cdn_frontdoor_security_policy`; link the firewall policy to the profile and associate it with the endpoint/route.

**Implementation (Terraform)**: Create `azurerm_cdn_frontdoor_firewall_policy` (mode Prevention or Detection, managed rule set such as Microsoft_DefaultRuleSet where available), then `azurerm_cdn_frontdoor_security_policy` referencing that firewall policy and the Front Door profile/domain. Apply to `/*` or the relevant path.

**Alternatives considered**: No WAF (simpler but not enterprise-ready); third-party WAF (adds cost and complexity; rejected for Azure-only constitution).

---

### 5.3 Managed Identity and Restrict Storage to Front Door Only

**Decision**: Use Managed Identity for Front Door → Storage access and restrict the Storage Account so only Front Door (or trusted Azure services) can reach it.

**Rationale**: solution.yaml already lists `security: Managed Identity`. Eliminates storage keys for origin access, reduces credential exposure, and satisfies "no unnecessary exposure." Restricting Storage (firewall / no public blob access for anonymous) so that only Front Door can read $web makes the origin non-public and enterprise-ready.

**Implementation (Terraform)**: (1) Enable system-assigned (or user-assigned) Managed Identity on the Front Door profile. (2) Grant that identity **Storage Blob Data Reader** on the Storage Account (RBAC). (3) Configure the Front Door origin to use Managed Identity authentication. (4) On the Storage Account: set `allow_nested_items_to_be_public = false`, enable firewall and restrict to "Allow Azure services on the trusted services list" or to Front Door's outbound IPs / Private Link if applicable. For static website with Front Door as sole reader, this keeps $web accessible only via Front Door.

**Alternatives considered**: Public read on $web (simpler, less secure); Storage keys for Front Door (not recommended; rejected).

---

**Summary**: The three Enterprise-ready configurations added to infrastructure are: **(1) HTTPS enforcement + minimum TLS 1.2**, **(2) WAF policy on Front Door**, **(3) Managed Identity for origin access + Storage restricted to Front Door only.** See [security.md](./security.md) and [infrastructure.yaml](./infrastructure.yaml) for the specification and resource list.
