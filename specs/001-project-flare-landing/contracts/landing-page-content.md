# Contract: Landing Page Content

**Feature**: 001-project-flare-landing  
**Type**: Static page content structure (no API)

## Purpose

Defines the required content and structure of the single landing page so that acceptance scenarios (spec.md) can be verified. This is a content contract, not an API or wire format.

## Required Content

| Element | Requirement | Source |
|---------|-------------|--------|
| Project name | Page MUST display “Project Flare” as the project name. | FR-002 |
| Primary message | Page MUST include a clear primary message or value statement. | FR-002 |
| Structure | Headings and body text MUST be present and readable; semantic structure (e.g., h1, main, paragraphs) for accessibility. | Acceptance scenario 2 |

## Optional

- Additional sections (e.g., features, contact).
- Images, favicon, CSS (no specific list required).
- Scripting is optional; core content MUST be visible without JavaScript (edge case in spec).

## Verification

- Manual: Open page in browser; confirm project name and primary message visible.
- Optional: HTML snapshot or link-check; no automated contract tests required for static content.
