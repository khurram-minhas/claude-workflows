---
name: security-review
description: Compact security checklist applied during analysis, plan review and self-review whenever a change touches authentication, authorization, user input, secrets, money, personal data, file handling or external calls. Findings that need a decision are escalated to a human, never resolved by the AI alone.
---

# Security review checklist

Trigger: the change touches any of — auth/session, permissions, user-supplied input, secrets or
config, payments/credits/quotas, personal data, file upload/download, outbound HTTP, SQL/NoSQL
queries, background jobs that act on behalf of users, admin tooling.

| Area | Check |
| --- | --- |
| Authentication | Endpoint/event requires the right identity; tokens validated server-side; no auth logic duplicated client-side only. |
| Authorization | Object-level checks (can *this* user act on *this* record), not just role checks. Admin paths gated. |
| Input | Validated at the boundary (schema/validator layer), not ad hoc in handlers; length/type/range; reject unknown fields where the framework allows. |
| Injection | Parameterised queries only; no string-built SQL/JQL/shell; templating auto-escapes. |
| Secrets | Nothing committed; read from env/secret store; not logged; not in error messages or client bundles. |
| Data exposure | Responses return the fields the caller needs, via a schema/serializer — not `dict(row)`. PII minimised in logs and analytics. |
| Money / quotas | Idempotent; server-authoritative; no client-provided prices or balances; race on double-submit considered. |
| Files | Type/size limits; no path traversal; stored outside the web root or in object storage. |
| Outbound calls | Timeouts, allow-listed hosts where possible, no SSRF via user URLs. |
| Randomness | Crypto-grade for tokens/ids that must be unguessable; document non-crypto use for game/UI logic. |
| Dependencies | New packages checked against the manifest first; known-vulnerable versions avoided. |
| Logging / audit | Security-relevant actions logged with actor + target, without secrets. |

## Rules

- A security finding is 🔴 by default.
- Decisions with product or legal weight (what data to keep, who may see what, how long
  tokens live) are **escalated to a human** with options — never chosen by the AI.
- Claude Code's built-in `/security-review` (when available) is a good deeper pass on the pending
  branch; this checklist is the minimum that always runs.
