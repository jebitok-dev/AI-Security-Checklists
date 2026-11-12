# Web Application + AI Security Checklist

For web applications with an integrated AI layer covering the full stack from authentication to AI-specific attack surfaces.

Start with `core-ai/ai-guardrails.md` then work through this checklist.

---

## Table of Contents

1. [Authentication & Session Management](#1-authentication--session-management)
2. [Authorisation & Access Control](#2-authorisation--access-control)
3. [Input Validation & Injection](#3-input-validation--injection)
4. [Prompt Injection (AI Layer)](#4-prompt-injection-ai-layer)
5. [Output Encoding & XSS Prevention](#5-output-encoding--xss-prevention)
6. [Security Headers](#6-security-headers)
7. [API Security](#7-api-security)
8. [AI Context Isolation & Data Handling](#8-ai-context-isolation--data-handling)
9. [Error Handling & Information Disclosure](#9-error-handling--information-disclosure)
10. [Dependency & Supply Chain](#10-dependency--supply-chain)
11. [Logging, Monitoring & Incident Response](#11-logging-monitoring--incident-response)
12. [Agentic Features (Tools, Memory, Delegation)](#12-agentic-features-tools-memory-delegation)
13. [Pre-launch Checklist (Minimum Viable Security)](#pre-launch-checklist-minimum-viable-security)
14. [Attack Scenarios](#attack-scenarios)
15. [References](#references)

---

## 1. Authentication & Session Management

> Frameworks: `[OWASP-A07]` `[OWASP-A02]` `[NIST-GV]`

- [ ] **Rate limit login attempts per IP and per account** `[CRITICAL]` `[OWASP-A07]`
  > Credential stuffing is automated. Without rate limiting, an attacker can try millions of passwords silently.

- [ ] **Lock account after N failed login attempts** `[CRITICAL]` `[OWASP-A07]`
  > Combine with rate limiting. Alert the user on lockout don't let an attacker lock users out silently.

- [ ] **Enforce MFA prefer TOTP over SMS OTP** `[CRITICAL]` `[OWASP-A07]`
  > SMS OTP is vulnerable to SIM swap attacks. TOTP authenticator apps are significantly more resilient.

- [ ] **Regenerate session token after login** `[CRITICAL]` `[OWASP-A07]`
  > Prevents session fixation. The pre-login session ID must be invalidated and replaced on successful authentication.

- [ ] **Invalidate session token on logout** `[CRITICAL]` `[OWASP-A07]`
  > Server-side invalidation not just clearing the client cookie. Old tokens must not remain valid.

- [ ] **Short session expiry with re-authentication for sensitive actions** `[STRIDE]` `[OWASP-A07]`
  > Long-lived sessions increase the window for session hijacking. Require re-auth for actions like changing email, password, or payment details.

- [ ] **Bind session to device fingerprint or IP where feasible** `[STRIDE]` `[OWASP-A07]`
  > Makes stolen session tokens less useful if used from a different context.

- [ ] **CAPTCHA or proof-of-work on registration and OTP triggers** `[STRIDE]` `[OWASP-A07]`
  > Prevents automated account creation and OTP flooding.

---

## 2. Authorisation & Access Control

> Frameworks: `[OWASP-A01]` `[LLM06]` `[LLM08]`

- [ ] **Enforce authorisation server-side on every request** `[CRITICAL]` `[OWASP-A01]`
  > Never trust the client to enforce access control. Every API call must be validated against the authenticated user's permissions.

- [ ] **Roles assigned and stored server-side only** `[CRITICAL]` `[STRIDE]` `[OWASP-A01]`
  > Never accept a role claim from a request body, JWT payload without server-side validation, or URL parameter.

- [ ] **Test horizontal privilege escalation can User A access User B's data?** `[CRITICAL]` `[STRIDE]` `[OWASP-A01]`
  > IDOR (Insecure Direct Object Reference) is consistently one of the most common and highest-impact web vulnerabilities. Test it explicitly for every resource type.

- [ ] **Test vertical privilege escalation can a low-privilege user reach admin endpoints?** `[CRITICAL]` `[STRIDE]` `[OWASP-A01]`
  > Discover admin endpoints via JS source, API fuzzing, or documentation then verify they reject non-admin tokens.

- [ ] **Non-sequential, non-enumerable IDs for all resources** `[STRIDE]` `[OWASP-A01]`
  > Sequential integer IDs make IDOR trivial. Use UUIDs or equivalent.

- [ ] **Remove or gate all debug and development endpoints before launch** `[CRITICAL]` `[OWASP-A05]`
  > Dev endpoints commonly bypass auth, expose internal state, or accept privileged operations without proper controls.

---

## 3. Input Validation & Injection

> Frameworks: `[OWASP-A03]` `[OWASP-A10]` `[LLM01]`

- [ ] **Validate all inputs server-side never trust client-side validation alone** `[CRITICAL]` `[OWASP-A03]`
  > Client-side validation is a UX feature, not a security control. It is trivially bypassed with Burp Suite or curl.

- [ ] **Parameterised queries for all database operations no string concatenation** `[CRITICAL]` `[OWASP-A03]`
  > SQL injection remains widespread. Parameterised queries eliminate the class entirely regardless of input content.

- [ ] **Whitelist accepted content types and file extensions on all upload endpoints** `[CRITICAL]` `[OWASP-A03]` `[OWASP-A08]`
  > Validate file type by content (magic bytes), not extension. A `.jpg` file can contain a PHP webshell.

- [ ] **Enforce file size limits server-side on all upload endpoints** `[STRIDE]` `[OWASP-A03]`
  > Client-side limits are bypassed in seconds. Server-side limits prevent storage exhaustion and memory-based DoS.

- [ ] **Scan uploaded files before storage or processing** `[CRITICAL]` `[OWASP-A08]`
  > Malware in uploaded files can execute when processed by the server. Scan before storage, not after.

- [ ] **Limit accepted URL schemes in any field that accepts URLs** `[AI THREAT]` `[STRIDE]` `[OWASP-A10]`
  > `file://`, `gopher://`, and internal IP ranges in URL fields are SSRF vectors. Allowlist `https://` to external domains only.

---

## 4. Prompt Injection (AI Layer)

> Frameworks: `[LLM01]` `[OWASP-A03]` `[ATLAS-T0051]`
> See `core-ai/ai-guardrails.md`: Section 1 for the full prompt injection checklist.

Additional web-app-specific items:

- [ ] **Treat every text input field as a potential injection point if it reaches the AI** `[CRITICAL]` `[AI THREAT]` `[LLM01]` `[ATLAS-T0051]`
  > Search boxes, contact forms, profile fields, comments any field whose content is passed to an AI model is an injection surface.

- [ ] **Test AI behaviour with adversarial inputs in every user flow** `[AI THREAT]` `[LLM01]` `[ATLAS-T0051]`
  > Don't just test the happy path. Inject prompt manipulation payloads into every field that feeds the AI and verify the output is correct.

- [ ] **Log and alert when AI output contains unexpected structural patterns** `[AI THREAT]` `[LLM01]` `[LLM06]`
  > Outputs that contain instruction-like language, system prompt fragments, or other users' data are indicators of a successful injection.

---

## 5. Output Encoding & XSS Prevention

> Frameworks: `[LLM02]` `[OWASP-A03]` `[OWASP-A05]`

- [ ] **Encode all output rendered in HTML context** `[CRITICAL]` `[OWASP-A03]`
  > HTML-encode `<`, `>`, `"`, `'`, `&` in any user-controlled or AI-generated content rendered in the browser.

- [ ] **Content Security Policy (CSP) header configured and tested** `[CRITICAL]` `[OWASP-A05]`
  > A correctly configured CSP limits the impact of XSS by blocking inline scripts and restricting script sources.

- [ ] **AI-generated content treated as untrusted and encoded before rendering** `[CRITICAL]` `[AI THREAT]` `[LLM02]` `[OWASP-A03]`
  > An attacker who can influence AI output can inject HTML or JavaScript into your UI if AI output is rendered without encoding.

- [ ] **X-Frame-Options or `frame-ancestors` CSP directive set** `[STRIDE]` `[OWASP-A05]`
  > Prevents clickjacking your app being embedded in an attacker-controlled iframe.

- [ ] **No `innerHTML` assignment with unescaped content** `[CRITICAL]` `[OWASP-A03]`
  > `innerHTML` with user-controlled or AI-generated content is a direct XSS vector regardless of other controls.

---

## 6. Security Headers

> Frameworks: `[OWASP-A05]` `[OWASP-A02]`

- [ ] **HTTPS enforced sitewide with HSTS header** `[CRITICAL]` `[OWASP-A02]` `[OWASP-A05]`
  > `Strict-Transport-Security: max-age=31536000; includeSubDomains`. Prevents protocol downgrade attacks.

- [ ] **Content-Security-Policy configured** `[CRITICAL]` `[OWASP-A05]`
  > Start restrictive and open up as needed. A permissive CSP provides no protection.

- [ ] **X-Content-Type-Options: nosniff** `[STRIDE]` `[OWASP-A05]`
  > Prevents MIME type sniffing the browser executing a response as a different content type than declared.

- [ ] **Referrer-Policy configured** `[STRIDE]` `[OWASP-A05]`
  > Controls how much information is sent in the Referer header. Sensitive URLs (reset tokens, session IDs in query strings) should not leak via Referer.

- [ ] **Permissions-Policy configured** `[STRIDE]` `[OWASP-A05]`
  > Restrict access to browser features (camera, microphone, geolocation) your app does not use.

- [ ] **No sensitive data in response headers** `[STRIDE]` `[OWASP-A05]`
  > Server version, framework version, internal hostnames strip these from responses.

---

## 7. API Security

> Frameworks: `[OWASP-A01]` `[OWASP-A05]` `[LLM04]` `[LLM10]` `[ATLAS-T0040]`

- [ ] **Authentication required on every API endpoint no unauthenticated paths to sensitive data** `[CRITICAL]` `[OWASP-A01]` `[OWASP-A07]`
  > Map every API route and verify auth is enforced. Unauthenticated endpoints are commonly left exposed after refactors.

- [ ] **Rate limit all API endpoints** `[CRITICAL]` `[STRIDE]` `[LLM04]` `[ATLAS-T0034]`
  > Both authenticated and unauthenticated. Per-user, per-IP, and global limits.

- [ ] **API returns only fields the requesting user is authorised to see** `[CRITICAL]` `[STRIDE]` `[OWASP-A01]`
  > Mass assignment and over-fetching returning more fields than needed leaks data even if the endpoint is authenticated.

- [ ] **Versioned API with deprecated versions disabled** `[STRIDE]` `[OWASP-A06]`
  > Old API versions often lack the security controls of newer ones. Disable them, don't just stop documenting them.

- [ ] **Request size limits enforced** `[STRIDE]` `[LLM04]`
  > Unbounded request bodies enable memory exhaustion. Set limits at the API gateway level.

- [ ] **CORS policy explicitly configured not wildcard** `[CRITICAL]` `[OWASP-A05]`
  > `Access-Control-Allow-Origin: *` on authenticated endpoints enables cross-origin attacks. Allowlist specific origins.

---

## 8. AI Context Isolation & Data Handling

> Frameworks: `[LLM06]` `[LLM08]` `[OWASP-A01]` `[ATLAS-T0024]`
> See `core-ai/ai-guardrails.md`: Sections 2, 3, 4 for the full AI context checklist.

Additional web-app-specific items:

- [ ] **Each user's AI session is scoped to their authenticated identity** `[CRITICAL]` `[AI THREAT]` `[LLM06]` `[OWASP-A01]`
  > The AI context for User A must be bound to User A's session token. Verify this cannot be bypassed by manipulating session parameters.

- [ ] **AI model cannot access data stores directly mediated via application layer** `[CRITICAL]` `[AI THREAT]` `[LLM08]` `[OWASP-A01]`
  > The application layer enforces authorisation. If the AI can query a database directly, it bypasses your access controls entirely.

- [ ] **Retrieval-augmented generation (RAG) results filtered by user permissions before being passed to AI** `[CRITICAL]` `[AI THREAT]` `[LLM06]` `[LLM08]` `[OWASP-A01]`
  > If your AI retrieves documents from a shared store, verify that users can only retrieve documents they are authorised to see even via the AI layer.

---

## 9. Error Handling & Information Disclosure

> Frameworks: `[OWASP-A05]` `[LLM06]`

- [ ] **Generic error messages to users in production detailed errors to server logs only** `[CRITICAL]` `[OWASP-A05]`
  > Stack traces, database query structures, and internal file paths in error responses are reconnaissance gold.

- [ ] **No sensitive data in URLs use POST body or headers** `[STRIDE]` `[OWASP-A05]`
  > Session tokens, reset tokens, or API keys in query strings appear in server logs, browser history, and Referer headers.

- [ ] **No secrets in client-side code or HTML source** `[CRITICAL]` `[OWASP-A02]`
  > API keys, internal endpoints, and credentials embedded in JavaScript or HTML are publicly readable.

- [ ] **No verbose debug information in production responses** `[STRIDE]` `[OWASP-A05]`
  > Debug modes, verbose logging, and development flags left enabled in production are a common source of unintended disclosure.

---

## 10. Dependency & Supply Chain

> Frameworks: `[OWASP-A06]` `[OWASP-A08]` `[LLM05]` `[ATLAS-T0010]`

- [ ] **All dependencies pinned to specific versions** `[CRITICAL]` `[OWASP-A06]` `[LLM05]`
  > Unpinned dependencies are a supply chain risk. A malicious update to a dependency can affect your application on the next install.

- [ ] **Dependency CVE scan run before every release** `[CRITICAL]` `[OWASP-A06]`
  > `npm audit`, `pip-audit`, `bundler-audit` pick the right tool for your stack and run it in CI.

- [ ] **No secrets in version control** `[CRITICAL]` `[OWASP-A02]`
  > Scan with `trufflehog`, `git-secrets`, or `gitleaks` before any push to a shared or public repository.

- [ ] **Third-party scripts loaded from controlled sources only** `[STRIDE]` `[OWASP-A08]`
  > Scripts loaded from CDNs or third-party domains can be modified without your knowledge. Use Subresource Integrity (SRI) hashes.

- [ ] **Software Bill of Materials (SBOM) maintained** `[COMPLIANCE]` `[OWASP-A06]`
  > Know what is in your application. Required in some regulated environments and useful for rapid response to new CVEs.

---

## 11. Logging, Monitoring & Incident Response

> Frameworks: `[OWASP-A09]` `[NIST-MS]` `[NIST-MG]` `[ATLAS-T0006]`

- [ ] **Centralised logging for all authentication events** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A09]`
  > Logins, logouts, failures, password resets, MFA events all logged with timestamp, user ID, and IP.

- [ ] **Alerts on threshold breaches error rate spikes, unusual auth patterns, high AI query volume** `[STRIDE]` `[OWASP-A09]` `[ATLAS-T0006]`
  > Monitoring without alerting is not monitoring. Define thresholds and route alerts to a channel someone watches.

- [ ] **Incident response runbook documented and tested** `[CRITICAL]` `[COMPLIANCE]` `[NIST-MG]`
  > Who is contacted first? What is the breach notification timeline? Who has authority to take the system offline? Document it before you need it.

- [ ] **Log retention policy defined and enforced** `[COMPLIANCE]` `[OWASP-A09]`
  > Too short and you lose forensic evidence. Too long and you hold data you are not authorised to retain.

---

## 12. Agentic Features (Tools, Memory, Delegation)

> Frameworks: `[ASI01]`–`[ASI10]` `[LLM08]` `[LLM07]` `[OWASP-A01]` `[OWASP-A03]` `[ATLAS-T0051]`
> Applies only if the AI can call tools or APIs, store memory across sessions, run code, or hand work to other agents. A chat or RAG feature that only answers is covered by sections 1–11.
> See `frameworks/owasp-agentic-top-10.md` for the full ASI01–ASI10 checklist. This section covers what changes in a web app.

Once the AI can act, every tool becomes an API endpoint that the model calls on the user's behalf. A prompt injection is then no longer just wrong text on the screen; it can turn into a real action against a real account.

- [ ] **Every tool call is authorised as the requesting user, server-side, at the tool layer** `[CRITICAL]` `[AI THREAT]` `[ASI03]` `[LLM08]` `[OWASP-A01]`
  > The agent usually runs under a service account that can reach far more than any one user. If the tool trusts the agent instead of re-checking the user's permissions, any user can reach any data by asking the agent (the "confused deputy" problem).

- [ ] **Tool arguments are validated like any request body** `[CRITICAL]` `[ASI02]` `[LLM07]` `[OWASP-A03]`
  > The model fills in tool arguments from text an attacker can influence. Parameterise SQL, never build shell commands from arguments, restrict file paths to an allowed directory, and allowlist URL targets.

- [ ] **Content the agent reads cannot trigger state-changing tools without user confirmation** `[CRITICAL]` `[AI THREAT]` `[ASI01]` `[LLM01]` `[ATLAS-T0051]`
  > Web pages, emails, support tickets, uploaded files and RAG chunks can all carry instructions. Reading is allowed; sending, deleting, paying, sharing or changing permissions after reading needs explicit user approval.

- [ ] **Each tool has a defined scope: allowed actions, targets, and maximum impact** `[CRITICAL]` `[ASI02]` `[LLM08]`
  > "Can send email" is not a scope. "Can send email to the requesting user's own address, max 5 per session" is. Prefer read-only tools; split read and write into separate tools so write can be gated.

- [ ] **Confirmation screens show the real action, built from the tool-call arguments** `[CRITICAL]` `[ASI09]`
  > Show recipient, amount, file, record and permission exactly as the tool will receive them. Do not show the model's own explanation of why the action is fine; that text is what an attacker controls.

- [ ] **Approvals are logged with who approved, when, and the exact arguments approved** `[COMPLIANCE]` `[ASI09]` `[OWASP-A09]`
  > If the arguments executed differ from the arguments approved, that should be impossible, or at minimum alert.

- [ ] **Agent memory is scoped per user and tenant, with provenance on every write** `[CRITICAL]` `[AI THREAT]` `[ASI06]` `[LLM06]` `[OWASP-A01]`
  > Persistent memory turns a one-off injection into a permanent one. Record where each memory came from (user typed it vs. agent read it in a document), let users view and delete their memories, and expire low-trust entries.

- [ ] **Tool servers and plugins (including MCP servers) are allowlisted and version-pinned** `[CRITICAL]` `[ASI04]` `[LLM05]` `[OWASP-A06]`
  > No user-supplied tool endpoints. A tool's description is also prompt input, so a changed tool description can change agent behaviour without any code change. Review tool updates like dependency updates.

- [ ] **Code execution features run in a sandbox with no credentials and no internal network access** `[CRITICAL]` `[AI THREAT]` `[ASI05]` `[LLM02]` `[ATLAS-T0050]`
  > No root, no cloud metadata endpoint (`169.254.169.254`), no access to internal services, short time and memory limits, and a fresh environment per session.

- [ ] **Per-session caps on tool calls, loop iterations, and spend** `[CRITICAL]` `[ASI08]` `[LLM04]` `[ATLAS-T0034]`
  > Agents retry and loop. Without a hard ceiling, one bad tool result or one injected instruction can make thousands of calls, at the user's or your expense.

- [ ] **Output from other agents and tools is treated as untrusted input** `[AI THREAT]` `[ASI07]` `[LLM01]`
  > In multi-agent setups, a sub-agent's reply is content, not instructions. Authenticate agent-to-agent calls and do not let an upstream agent's text widen a downstream agent's permissions.

- [ ] **Every agent action is logged with the five facts and can be stopped** `[CRITICAL]` `[ASI10]` `[OWASP-A09]`
  > Who asked, which tool, which arguments, which credential, and what triggered it (user message, scheduled job, another agent). Have a per-agent kill switch that also revokes the agent's credentials. See `core-ai/observability.md`.

---

## Pre-launch Checklist (Minimum Viable Security)

Before any real users touch the system:

- [ ] All CRITICAL items above addressed `[CRITICAL]`
- [ ] Prompt injection tested on every field that reaches the AI `[CRITICAL]` `[AI THREAT]` `[LLM01]`
- [ ] Cross-tenant AI context leakage tested `[CRITICAL]` `[AI THREAT]` `[LLM06]`
- [ ] Dependency CVE scan clean no critical or high findings `[CRITICAL]` `[OWASP-A06]`
- [ ] No secrets in the codebase or version control `[CRITICAL]` `[OWASP-A02]`
- [ ] Security headers configured and verified `[CRITICAL]` `[OWASP-A05]`
- [ ] Privacy notice and consent mechanism live `[CRITICAL]` `[COMPLIANCE]` `[NIST-GV]`
- [ ] AI use disclosed to users `[CRITICAL]` `[COMPLIANCE]` `[LLM09]` `[NIST-GV]`
- [ ] Logging verified, not just configured: a test login, a test AI prompt, and a test sensitive-data access each appear in central logs with timestamp, user ID, and session ID, and an alert fires on a test threshold breach `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A09]`
- [ ] Incident response runbook written `[CRITICAL]` `[COMPLIANCE]` `[NIST-MG]`
- [ ] Debug endpoints removed or gated `[CRITICAL]` `[OWASP-A05]`
- [ ] If the AI can act (tools, memory, code, delegation): all section 12 CRITICAL items addressed, and indirect injection tested against every tool-enabled flow `[CRITICAL]` `[AI THREAT]` `[ASI01]` `[ASI02]`

---

## Attack Scenarios

See `web-app-ai/attack-scenarios.md` for real-world examples of how each vulnerability class plays out in an AI-integrated web application.

---

## References

- [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/)
- [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [NIST AI Risk Management Framework](https://www.nist.gov/system/files/documents/2023/01/26/AI%20RMF%201.0.pdf)
