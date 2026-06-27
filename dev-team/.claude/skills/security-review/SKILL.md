---
name: security-review
description: OWASP-grade security review checklist for a code diff and its dependency/config changes. Use when reviewing a diff for vulnerabilities — injection, broken access control, secrets, unsafe deserialization, SSRF, path traversal, insecure crypto, and vulnerable dependencies. Defers concrete scanner commands to the per-language standards skill.
---

# security-review

Adversarial pass over a diff. State the *concern*; the per-language standards skill supplies the
concrete vuln-scanner command for the stack.

## Checklist (OWASP Top 10 grain)
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

## How to report
Classify each finding `critical | high | medium | low`, tie it to `file:line`, give a one-line
exploit sketch and a concrete remediation. Ground every claim in code evidence or a verified
advisory — never assert a CVE or risk you have not confirmed. Mark genuine uncertainty as
"needs human security confirmation". This skill governs *what to check*; it never suppresses a
required finding.
