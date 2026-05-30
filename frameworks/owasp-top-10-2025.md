# OWASP Top 10 for Web Applications - 2025

Reference for the `[OWASP-A0X]` tags used across the checklists. The 2025 release consolidates a decade of telemetry from web application breaches and bug bounty data.

Source: [OWASP Top 10](https://owasp.org/Top10/)

Note: As of the 2026 cutoff for this repo, the 2025 release is the current canonical list. Categories below reflect the 2025 ordering. If you are reading this after a later release supersedes 2025, re-check the OWASP source.

---

## Table of Contents

1. [A01: Broken Access Control](#a01-broken-access-control)
2. [A02: Cryptographic Failures](#a02-cryptographic-failures)
3. [A03: Injection](#a03-injection)
4. [A04: Insecure Design](#a04-insecure-design)
5. [A05: Security Misconfiguration](#a05-security-misconfiguration)
6. [A06: Vulnerable and Outdated Components](#a06-vulnerable-and-outdated-components)
7. [A07: Identification and Authentication Failures](#a07-identification-and-authentication-failures)
8. [A08: Software and Data Integrity Failures](#a08-software-and-data-integrity-failures)
9. [A09: Security Logging and Monitoring Failures](#a09-security-logging-and-monitoring-failures)
10. [A10: Server-Side Request Forgery (SSRF)](#a10-server-side-request-forgery-ssrf)
11. [Cross-reference: Web ↔ LLM](#cross-reference)

---

## A01: Broken Access Control

Authorisation enforcement failures: IDOR, missing function-level access control, JWT role tampering, path traversal that bypasses access checks. The most consistently exploited category in production.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §2 (Authorisation & Access Control, full section)
- `web-app-ai/checklist.md` §8 (RAG results filtered by user permissions)
- `web-app-ai/attack-scenarios.md` Scenarios 5, 6, 7, 8
- `health-ai/checklist.md` §5 (Access Control in Clinical Workflows)

---

## A02: Cryptographic Failures

Weak, missing, or misused cryptography. Plain-text storage of sensitive data, weak TLS configuration, predictable tokens, hardcoded keys.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §6 (HTTPS / HSTS)
- `web-app-ai/checklist.md` §9 (no secrets in client-side code)
- `health-ai/checklist.md` §1 (encryption at rest and in transit)

---

## A03: Injection

SQL, NoSQL, OS command, LDAP injection and now prompt injection into LLMs as a first-class injection class.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §3 (Input Validation & Injection)
- `web-app-ai/checklist.md` §4 (Prompt Injection)
- `core-ai/ai-guardrails.md` §1 (Prompt Injection Defences)

---

## A04: Insecure Design

Architectural flaws: missing threat modelling, missing security controls at design time, business logic vulnerabilities that no code review can catch.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §5 (Agentic AI permission design)
- `core-ai/ai-guardrails.md` §8 (Compliance & Governance Baseline incident response runbook before launch)
- `fintech-ai/checklist.md` §1 (AI in Financial Decision Pipelines - design-level controls)

---

## A05: Security Misconfiguration

Default credentials, verbose errors, missing security headers, exposed admin interfaces, overly permissive CORS.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §6 (Security Headers)
- `web-app-ai/checklist.md` §7 (CORS policy explicit, not wildcard)
- `web-app-ai/checklist.md` §9 (Error Handling & Information Disclosure)
- `web-app-ai/checklist.md` §2 (Remove or gate debug endpoints)

---

## A06: Vulnerable and Outdated Components

Unpatched dependencies, unpinned versions, abandoned libraries, AI models pulled from unverified sources.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §10 (Dependency & Supply Chain)
- `core-ai/ai-guardrails.md` §3 (Pin model versions, CVE scan)
- `web-app-ai/attack-scenarios.md` Scenario 9

---

## A07: Identification and Authentication Failures

Weak password policy, missing MFA, session fixation, predictable session IDs, credential stuffing without rate limit.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §1 (Authentication & Session Management, full section)
- `fintech-ai/checklist.md` §4 (High-Value Account Protection)

---

## A08: Software and Data Integrity Failures

Unsigned updates, untrusted CI/CD pipelines, deserialisation of untrusted data, AI model integrity not verified.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §3 (AI Supply Chain & Model Integrity)
- `core-ai/ai-guardrails.md` §4 (Immutable audit log of AI inputs and outputs)
- `web-app-ai/checklist.md` §10 (SBOM, dependency pinning)
- `fintech-ai/checklist.md` §1 (AI output immutability)

---

## A09: Security Logging and Monitoring Failures

Insufficient logging, no alerting, no log integrity, logs lost during incident response.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §7 (Logging, Audit Trails & Repudiation)
- `web-app-ai/checklist.md` §11 (Logging, Monitoring & Incident Response)
- `fintech-ai/checklist.md` §1 (Audit trail for every AI-assisted financial decision)
- `health-ai/checklist.md` §2 (Override logging for clinician overrides)

---

## A10: Server-Side Request Forgery (SSRF)

The application fetches a URL supplied or influenced by an attacker. AI agents that fetch URLs on behalf of users are a new SSRF surface.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §3 (Limit accepted URL schemes)
- `core-ai/ai-guardrails.md` §5 (Block AI from making outbound requests to user-supplied URLs)

---

## Cross-reference

| Web (OWASP Top 10) | LLM equivalent | Notes |
|--------------------|----------------|-------|
| A01 Broken Access Control | LLM06, LLM08 | AI bypasses traditional auth if it queries data stores directly |
| A03 Injection | LLM01 | Prompt injection is a first-class injection class |
| A05 Misconfiguration | LLM02 | Output handling is a configuration choice |
| A06 Vulnerable Components | LLM05 | Models and ML libs are dependencies |
| A08 Integrity Failures | LLM03 | Training data poisoning is an integrity failure |
| A09 Logging Failures | LLM09 | Without logging you cannot detect overreliance harms |
| A10 SSRF | LLM07, LLM08 | Agentic AI URL-fetching is the new SSRF |
