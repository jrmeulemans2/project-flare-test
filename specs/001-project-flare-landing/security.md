# Security Specification: Project Flare Landing (Enterprise Ready)

**Feature**: 001-project-flare-landing  
**Date**: 2026-03-04  
**Alignment**: [spec.md](./spec.md) FR-003, FR-006; [solution.yaml](./solution.yaml) security: Managed Identity

## Overview

This document specifies three Azure security configurations added to the infrastructure so the Project Flare static landing (Azure Storage + Azure Front Door) meets **Enterprise Ready** expectations. All are implemented via Terraform and reflected in [infrastructure.yaml](./infrastructure.yaml).

---

## 1. HTTPS Enforcement and Minimum TLS 1.2

**Goal**: All traffic to the landing page is encrypted in transit; no HTTP and no weak TLS.

| Item | Specification |
|------|----------------|
| **Scope** | Azure Front Door (public edge). |
| **HTTPS** | Accept only HTTPS on frontend endpoints; redirect HTTP → HTTPS. |
| **TLS** | Minimum TLS version **1.2** (TLS 1.0/1.1 disabled). |
| **Terraform** | Front Door route/endpoint: `accepted_protocols = ["Https"]`; custom domain / TLS policy with minimum TLS 1.2. |

**References**: Spec FR-003, SC-002; [research.md §5.1](./research.md#51-https-enforcement-and-minimum-tls).

---

## 2. WAF Policy on Front Door

**Goal**: Protect the landing page at the edge with a Web Application Firewall (OWASP-oriented and optional rate limiting).

| Item | Specification |
|------|----------------|
| **Scope** | Azure Front Door profile (Standard/Premium). |
| **Resources** | `azurerm_cdn_frontdoor_firewall_policy` + `azurerm_cdn_frontdoor_security_policy`. |
| **Mode** | Prevention (recommended) or Detection. |
| **Rules** | Use Microsoft managed rule set where available (e.g. Microsoft_DefaultRuleSet on Premium); optional custom rules (rate limit, IP allow/block). |
| **Association** | Security policy applied to Front Door endpoint/route (e.g. path `/*`). |

**References**: [research.md §5.2](./research.md#52-waf-policy-on-front-door).

---

## 3. Managed Identity and Storage Restricted to Front Door Only

**Goal**: No storage account keys for origin access; Storage is not publicly readable—only Front Door can read $web.

| Item | Specification |
|------|----------------|
| **Front Door** | Enable **Managed Identity** (system-assigned or user-assigned) on the Front Door profile. |
| **RBAC** | Grant the Front Door identity **Storage Blob Data Reader** on the Storage Account. |
| **Origin** | Configure Front Door origin to use **Managed Identity** authentication (no key). |
| **Storage** | `allow_nested_items_to_be_public = false`; enable **network rules** (firewall): restrict to “Allow Azure services on the trusted services list” or to Front Door (e.g. Private Link / allowed Microsoft services as per Azure docs). So $web is only reachable via Front Door. |

**References**: solution.yaml `security: Managed Identity`; [research.md §5.3](./research.md#53-managed-identity-and-restrict-storage-to-front-door-only).

---

## Infrastructure Summary

The following are added or updated in [infrastructure.yaml](./infrastructure.yaml) and implemented in Terraform:

1. **Front Door**: HTTPS-only, minimum TLS 1.2, Managed Identity enabled, origin using Managed Identity.
2. **WAF**: Firewall policy + security policy attached to Front Door and applied to the default route.
3. **Storage**: Firewall/network restriction so only Front Door (via Managed Identity or trusted services) can access the account; no public anonymous blob access required for $web.

---

## Compliance Notes

- **Constitution**: Azure-only (I), East US (II), Terraform IaC (III), static-first (IV), simplicity (V)—all preserved; security additions are documented and traceable.
- **Spec**: FR-003 (secure channel) and FR-006 (security best practices) are satisfied; no new collection of personal data.
