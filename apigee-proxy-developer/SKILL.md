---
name: apigeex-proxy-developer
description: >-
  Develops, designs, and manages Apigee X API Proxies and Shared Flows. 
  Use when creating new proxies, modifying existing flow logic, implementing 
  security policies, or troubleshooting deployment failures. 
  Don't use for general Cloud Infrastructure (use gcp skill) or 
  analytics queries (use apigee-analytics skill).
---

# Apigee Proxy Developer

Opinionated guidance for designing and managing Apigee API Proxies using `apigeecli`.

## Workflow & Safety
- **Never deploy automatically:** Only `upload` proxy bundles. Manual deployment is mandatory to prevent accidental production impact.
- **No manual zipping:** Do NOT attempt to create a `.zip` bundle manually. Use the `apigeecli` direct folder upload method instead.
- **Self-Improvement:** If an upload fails due to a configuration error, you MUST update the **Policy Gotchas** section in this file for common issues, or `references/special-cases.md` for rare/project-specific ones.
- **Verification:** Always verify tool availability before starting:
  `command -v apigeecli`

## Implementation Patterns

### 1. Policy Gotchas (High Signal)
- **Quota Policy:**
    - `type="calendar"` REQUIRES a `<StartTime>`.
    - Prefer `type="default"` unless calendar-alignment is explicitly requested.
    - When using `UseQuotaConfigInAPIProduct`, the `<Allow>` element must contain the count as text: `<Allow>100</Allow>`, NOT an attribute.
- **Fault Handling:** Always implement a `DefaultFaultRule` **to ensure consistent, branded error responses and mask internal backend implementation details.**

### 2. Routing & Logic
- **PreFlow/PostFlow:** **Enforce** security (OAuth/APIKey) in PreFlow and **capture** analytics/logging in PostFlow.
- **Conditionals:** **Filter** traffic using `request.verb`, `proxy.pathsuffix`, and `client.ip` for granular control.
- **Chaining:** **Utilize** `LocalTargetConnection` when chaining proxies in the same environment **to minimize latency and internal data egress costs.**

## Tooling Quick-Reference
- **Upload Bundle:** `apigeecli apis create bundle -o {org} -n {name} -f ./{proxy_dir}/apiproxy --default-token` (Note: `-f` points to the `apiproxy` directory).
- **Export Bundle:** `apigeecli apis export -n {name} -o {org} --default-token`

## References
- [Special Cases & Lessons Learned](file://references/special-cases.md)
- [Google Cloud Documentation Links](file://references/google-docs.md)
- [Apigee Antipatterns](file://references/antipatterns-intro.md)
