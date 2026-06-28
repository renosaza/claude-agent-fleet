---
name: security-review
description: OWASP-grade security review checklist for a code diff (Mode A) and a single-attack-class whole-repo audit methodology (Mode B). Use when reviewing a diff for vulnerabilities — injection, broken access control, secrets, unsafe deserialization, SSRF, path traversal, insecure crypto, vulnerable dependencies — or when hunting one of nine named attack classes across an entire repo with adversarial validation. Defers concrete scanner commands to the per-language standards skill.
---

# security-review

Adversarial pass over code. **Mode A** = a diff review (the OWASP checklist below). **Mode B** =
a deep audit scoped to ONE attack class hunted across the whole repo (the nine classes + the
validation gate + the report structure below). State the *concern*; the per-language standards
skill supplies the concrete vuln-scanner command for the stack.

## Mode A — Diff checklist (OWASP Top 10 grain)
- **Injection** — SQL, command, template, LDAP. Is untrusted input ever concatenated into a
  query/command/template? Require parameterized queries and safe APIs.
- **Broken access control / authz** — does the change expose data or actions without the right
  ownership/role check? Look for missing checks on new endpoints/handlers.
- **Secrets** — API keys, tokens, passwords, private keys in code, config, or fixtures. Flag any
  literal credential; require it to move to env/secret store.
- **Unsafe deserialization** — pickle/yaml.load/`eval`/native deserialize on untrusted input.
- **SSRF / path traversal** — user-controlled URLs or file paths reaching network/file I/O
  without an allowlist or canonicalization.
- **Insecure crypto / transport** — weak hashes for secrets, homemade crypto, disabled TLS
  verification, predictable randomness for security tokens.
- **Vulnerable dependencies** — run the stack's scanner (see the per-language skill) and read the
  advisories for newly added/updated packages.
- **Input validation at boundaries** — validate at the system edge (user input, external API);
  do not trust internal callers as a substitute.

## Mode B — Deep audit attack classes
For a whole-repo audit, the hunter is scoped to exactly ONE of these nine classes per dispatch and
hunts it across the entire repo. The orchestrator runs one hunter per class in parallel. Each class
lists its key hunting questions — answer them with evidence, not intuition.

1. **Injection** — Does untrusted input reach a SQL / HTML / shell / template / file-path /
   deserialization sink? Is there indirect or *stored* injection (input persisted, executed later)?
   Can injection ride in via field names, keys, headers, or metadata — not just values?
2. **Access Control** — Is there an alternative path to the same state change with weaker perms?
   Can a request field (role, owner_id, tenant) override the permission check? Are there endpoints
   that authenticate but never authorize? Do bulk / batch / export operations enforce per-item
   permissions, or only one gate at the top?
3. **Resource & File Handling** — Can a request read or write outside intended dirs via symlinks or
   encoded sequences? Does it fetch attacker-controlled URLs or follow redirects into the internal
   network (SSRF)? Are temp files, archive extraction, or deserialization unsafe? Any TOCTOU race
   between check and use?
4. **Cryptography & Secrets** — Is randomness sufficient for tokens / keys / nonces (CSPRNG, not
   `rand()`)? Are secrets hardcoded or leaked in logs, errors, or responses? Are KDF / HMAC / nonce
   usage correct? Does a crypto-failure fallback silently disable the protection?
5. **Business Logic** — Can workflow steps be skipped, reversed, or replayed? Do concurrent
   operations reach an invalid state (double-spend, lost update)? Any numeric overflow or
   type-coercion bug? Is expiry / clock-skew handled? What is the security posture when a dependency
   is unavailable — does it fail open?
6. **Feature Abuse & Data Leakage** — Does export / backup leak data above the caller's access
   level or recover deleted content? Can import overwrite records or bypass validation? Do search /
   filter reveal hidden content via ordering or error differences? Is enumeration possible via error
   messages, timing, or status codes? Do preview tokens, drafts, or caches expose private content?
7. **Chained Attacks & Trust Boundaries** — Can low-priv actions combine into a high-priv result
   (info-disclosure + IDOR)? Does a component blindly trust upstream validation? Is a field safe in
   one context dangerous in another? Are tokens / scopes broader than intended? Any check-then-act
   race across a boundary?
8. **Wildcard** — Look at the strangest, experimental, or unused code. Are there API calls the code
   supports but the frontend never uses? Undocumented endpoints or params? Features mixed together
   that were never designed to interact?
9. **Obvious Things** — Hardcoded passwords / keys / tokens? `TODO`/`FIXME` referencing skipped
   security? Debug mode enableable in prod? Functional test credentials in prod? Unprotected admin
   or debug endpoints? Missing `HttpOnly` / `Secure` / `SameSite` cookie attrs? Wildcard or
   over-permissive CORS? Stack traces or SQL errors returned in responses?

## Mode B — Adversarial validation (the 5 tests)
A candidate is NOT a finding until it survives all five. Kill false positives aggressively, but
don't kill real findings.
1. **Exploitation** — Can you construct the exact triggering input (request / payload / sequence)?
   If you cannot write it down, it is not a finding.
2. **Impact** — What does the attacker actually obtain? Learning a field name or merely triggering
   an error is LOW at best. Be honest about the prize.
3. **Baseline / mitigation** — Does a middleware, DB constraint, framework default, or upstream
   check already prevent it? Verify the mitigation is actually wired in, not just present elsewhere.
4. **Parser / runtime behavior** — Verify against the actual spec or implementation (and `context7`
   for library/parser behavior), not intuition about how it "should" parse or coerce.
5. **Completeness** — Did you omit a prerequisite (auth, a feature flag, a specific config) that the
   attack silently assumes? List every precondition.

## Mode B — Report structure (Markdown, never JSON)
Produce a self-contained Markdown section for your one class. Brevity rule: if the report is longer
than the codebase deserves, you're padding.
- **Executive summary** — one honest paragraph: what you hunted, what you found, the posture for
  this class.
- **Findings table** — `Severity | Title | one-line summary`.
- **Per-finding detail** — `file:line`, the concrete attack scenario / exact triggering request,
  the attacker impact, and a concrete remediation.
- **Hardening notes (NOT findings)** — defense-in-depth that isn't exploitable today; clearly
  separated so it is not mistaken for a vulnerability.
- **Positive patterns** — what the code does well for this class; this calibrates trust.
If nothing survived validation, state exactly: `No exploitable vulnerabilities found in <class>.`
and include any Hardening notes / Positive patterns you have.

## How to report
Classify each finding `critical | high | medium | low` (severity = likelihood × impact), tie it to
`file:line`, give a one-line exploit sketch and a concrete remediation. Ground every claim in code
evidence or a verified advisory — never assert a CVE or risk you have not confirmed. Mark genuine
uncertainty as "needs human security confirmation". This skill governs *what to check*; it never
suppresses a required finding.
