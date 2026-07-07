---
name: security-check
description: Full security and privacy audit. Use on demand or at the end of an implementation phase; CRITICAL findings block further work until resolved.
disable-model-invocation: true
---

# Security Check

Full security and privacy audit. Run on demand or at the end of an implementation phase.

CRITICAL findings block further implementation until resolved.

## Review Areas

Work through each area in order. Flag every finding — do not filter by perceived severity before listing.

**1. Data handling**
- What data is collected, stored, and transmitted?
- Is any of it personally identifiable (name, email, location, device ID, behavioral data)?
- Is there a defined data retention policy? Is it enforced in code?
- Is user consent obtained before data collection where required?

**2. Authentication and authorization**
- Are all routes and endpoints protected that should be?
- Is authentication enforced server-side, not only on the client?
- Are authorization checks present at the data layer, not only at the route layer?
- Are session tokens stored and transmitted securely — in httpOnly, Secure cookies rather than localStorage or other client-readable storage?

**3. Input validation**
- Is all user input validated before it is used, stored, or passed to another system?
- Is validation done server-side, not only client-side?
- Are file uploads validated for type and size?

**4. Injection risks**
- SQL injection: are queries parameterized or built using an ORM?
- Command injection: is user input ever passed to shell commands?
- Template injection: is user input rendered into templates without escaping?

**5. Client exposure**
- Are API keys, secrets, or credentials accessible from the client-side bundle?
- Are secrets present anywhere in git history, not only in the current working tree?
- Does the API return more data than the client needs?
- Are internal system details (stack traces, file paths, database structure) visible in error responses?
- Are security response headers configured, with a Content-Security-Policy that starts strict (no `unsafe-inline` or `unsafe-eval` by default)?

**6. Dependencies**
- Are any packages outdated with known vulnerabilities? Run the ecosystem's audit tool (`npm audit`, `pip-audit`, `cargo audit`, or equivalent).
- Are dependencies pinned to specific versions or ranges that allow silent upgrades?

## Output Format

For each finding:

```
[SEVERITY] — [Area]
Location: [file and line or endpoint]
Issue: [what the problem is]
Why it matters: [what an attacker or data breach could do with this]
Fix: [what to change]
```

Severity levels:
- CRITICAL: exploitable now, or exposes user data without consent. Blocks further work.
- HIGH: significant risk, should be resolved before the next milestone closes.
- MEDIUM: real risk, address before production release.

End with a summary: total findings by severity, and whether implementation may continue.
