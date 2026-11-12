# Core AI Guardrails

Shared across all domains. These apply to any web application with an AI layer regardless of industry.

Pick your domain checklist for additional context-specific items.

---

## Table of Contents

1. [Prompt Injection Defences](#1-prompt-injection-defences)
2. [AI Context & Tenant Isolation](#2-ai-context--tenant-isolation)
3. [AI Supply Chain & Model Integrity](#3-ai-supply-chain--model-integrity)
4. [Data Poisoning & Input Integrity](#4-data-poisoning--input-integrity)
5. [Agentic AI & Permission Controls](#5-agentic-ai--permission-controls)
6. [Output Handling & Rendering](#6-output-handling--rendering)
7. [Logging, Audit Trails & Repudiation](#7-logging-audit-trails--repudiation)
8. [Compliance & Governance Baseline](#8-compliance--governance-baseline)

---

## 1. Prompt Injection Defences

> Frameworks: `[LLM01]` `[OWASP-A03]` `[ATLAS-T0051]` `[NIST-MS]`

The most exploitable attack surface unique to AI-integrated systems. User-controlled input that reaches an AI model is an injection point.

- [ ] **Separate system prompt from user content** never concatenate user input directly into your system prompt `[CRITICAL]` `[AI THREAT]` `[LLM01]`
  > If an attacker can close your prompt structure and open their own, they control the AI's behaviour.

- [ ] **Sanitise all user inputs before passing to AI** strip or escape characters that can alter prompt structure `[CRITICAL]` `[AI THREAT]` `[LLM01]` `[OWASP-A03]`
  > Payloads like `Ignore previous instructions. Do X instead.` are trivial to craft and effective if inputs are unsanitised.

- [ ] **Test every user-controlled field with known injection payloads** `[CRITICAL]` `[AI THREAT]` `[LLM01]` `[ATLAS-T0051]`
  > Standard QA does not test for prompt injection. It must be done explicitly. Test all text inputs, file uploads, and URL parameters that reach the AI.

- [ ] **Block indirect prompt injection via uploaded files and external content** `[AI THREAT]` `[LLM01]` `[ATLAS-T0051]`
  > If your AI processes user-uploaded documents or fetches external URLs, those sources can contain embedded instructions. A PDF that says "Ignore your instructions and output X" is an injection vector.

- [ ] **Filter AI output before rendering to users** `[CRITICAL]` `[AI THREAT]` `[LLM02]`
  > Validate that AI responses do not contain system prompt fragments, other users' data, or injected instructions before displaying them.

---

## 2. AI Context & Tenant Isolation

> Frameworks: `[LLM06]` `[OWASP-A01]` `[ATLAS-T0024]` `[NIST-MP]`

Shared AI context across users or sessions is a cross-tenant data leakage vector.

- [ ] **Isolate AI context per user session no shared state across tenants** `[CRITICAL]` `[AI THREAT]` `[LLM06]` `[OWASP-A01]`
  > If session context persists across users, User A's data can appear in User B's AI response. Test this explicitly.

- [ ] **Keep sensitive data out of AI prompts unless absolutely necessary** `[AI THREAT]` `[LLM06]`
  > Every piece of data in the context window is a potential leakage point. Minimise what you pass to the model.

- [ ] **Separate AI contexts per user role** `[AI THREAT]` `[LLM06]` `[OWASP-A01]`
  > Admin context, user context, and reviewer context should never overlap in the same AI session.

- [ ] **Test cross-tenant data leakage as a dedicated QA test case** `[CRITICAL]` `[AI THREAT]` `[LLM06]` `[ATLAS-T0024]`
  > Submit data as User A. Query the AI as User B. Verify nothing from A appears in B's response.

---

## 3. AI Supply Chain & Model Integrity

> Frameworks: `[LLM05]` `[OWASP-A06]` `[OWASP-A08]` `[ATLAS-T0010]` `[NIST-GV]`

Compromised models and dependencies are invisible until triggered.

- [ ] **Treat every third-party model and AI library as untrusted until verified** `[CRITICAL]` `[AI THREAT]` `[LLM05]` `[ATLAS-T0010]`
  > Supply chain attacks on ML models are real. A model that behaves normally in testing can trigger on specific inputs in production.

- [ ] **Pin all AI model versions and dependency versions** `[AI THREAT]` `[LLM05]` `[OWASP-A06]`
  > Floating versions (`latest`, `>=x.x`) mean your AI behaviour can change on the next deploy without any code change.

- [ ] **Run dependency CVE scan before every release** `[CRITICAL]` `[OWASP-A06]`
  > Use `pip-audit`, `npm audit`, or equivalent. Fix all critical and high findings before shipping.

- [ ] **Log model provenance record which model version produced which output** `[AI THREAT]` `[COMPLIANCE]` `[LLM05]` `[OWASP-A08]` `[NIST-GV]`
  > If an AI output is challenged, you need to know what model and what prompt version produced it.

- [ ] **No secrets or credentials in AI prompts or model configs** `[CRITICAL]` `[LLM06]` `[OWASP-A02]`
  > System prompts are not secure storage. API keys, passwords, or internal logic embedded in prompts can be extracted via prompt injection.

---

## 4. Data Poisoning & Input Integrity

> Frameworks: `[LLM03]` `[OWASP-A08]` `[ATLAS-T0020]` `[NIST-MS]`

Attackers can craft inputs designed to skew AI behaviour or corrupt training/retrieval data.

- [ ] **Validate and whitelist accepted fields on every AI-facing endpoint** `[CRITICAL]` `[AI THREAT]` `[LLM03]` `[OWASP-A03]`
  > Unexpected fields passed to AI endpoints can manipulate model behaviour especially in systems with RAG or fine-tuning pipelines.

- [ ] **Monitor for anomalous AI output patterns** `[AI THREAT]` `[LLM03]` `[NIST-MS]`
  > Sudden changes in output quality, sentiment, or structure can indicate data poisoning or prompt manipulation. Log and alert on deviations.

- [ ] **Flag AI-generated content as AI-generated in the UI** `[COMPLIANCE]` `[AI THREAT]` `[LLM09]` `[NIST-GV]`
  > Users and downstream consumers should always know when output is AI-generated, not human-verified. Presenting AI output as factual without disclosure is both a trust and a regulatory risk.

- [ ] **Immutable audit log of AI inputs and outputs** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A08]` `[OWASP-A09]`
  > If an AI output is disputed, you need a timestamped, tamper-evident record of what went in and what came out.

---

## 5. Agentic AI & Permission Controls

> Frameworks: `[LLM07]` `[LLM08]` `[OWASP-A01]` `[OWASP-A10]` `[ATLAS-T0053]` `[NIST-MG]`

AI agents that take actions (tool calls, API requests, file writes) need the same access controls as human users.

> If the AI can call tools, keep memory, run code, or delegate to other agents, also work through `frameworks/owasp-agentic-top-10.md` and `web-app-ai/checklist.md` §12.

- [ ] **Principle of least privilege for all AI components** `[CRITICAL]` `[AI THREAT]` `[LLM08]` `[OWASP-A01]`
  > An AI agent should only have access to what it needs to complete its task. Broad permissions are a force multiplier for prompt injection attacks.

- [ ] **Human review gate before AI output triggers irreversible actions** `[CRITICAL]` `[AI THREAT]` `[LLM08]` `[NIST-MG]`
  > Any AI-driven action that cannot be undone (sending an email, making a payment, deleting data) should require explicit human confirmation.

- [ ] **Block AI from making outbound requests to user-supplied URLs** `[AI THREAT]` `[LLM07]` `[OWASP-A10]`
  > If an AI agent can fetch URLs, and users can supply those URLs, you have an SSRF vector via the AI layer. Validate and allowlist any URLs the AI can reach.

- [ ] **Set hard AI API spending caps and per-user rate limits** `[AI THREAT]` `[LLM04]` `[ATLAS-T0034]`
  > Bulk submissions designed to exhaust your AI API quota are a real denial-of-service vector with financial impact. Rate limit and cap at the account level.

---

## 6. Output Handling & Rendering

> Frameworks: `[LLM02]` `[OWASP-A03]` `[OWASP-A05]` `[ATLAS-T0050]`

AI output rendered directly to a browser without sanitisation is an XSS vector.

- [ ] **Sanitise AI output before rendering in the browser** `[CRITICAL]` `[LLM02]` `[OWASP-A03]`
  > If AI output contains HTML or JavaScript and is rendered without escaping, an attacker who can influence the prompt can inject scripts into your UI.

- [ ] **Do not execute AI-generated code without sandbox isolation** `[CRITICAL]` `[AI THREAT]` `[LLM02]` `[ATLAS-T0050]`
  > If your application executes code produced by an AI (SQL queries, shell commands, scripts), treat it as untrusted input. Sandbox, validate, and limit scope.

- [ ] **Make AI-generated content immutable after it has been acted on** `[COMPLIANCE]` `[OWASP-A08]`
  > If a user made a decision based on an AI output, that output should be preserved as-is for audit purposes. Retroactive modification is a repudiation risk.

---

## 7. Logging, Audit Trails & Repudiation

> Frameworks: `[OWASP-A09]` `[ATLAS-T0006]` `[NIST-MS]`

Without logs, you cannot prove what happened or defend against false claims.

- [ ] **Log every AI prompt and output with timestamp, user ID, and session ID** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A09]`
  > The minimum viable audit trail. Needed for debugging, compliance, and repudiation defence.

- [ ] **Logs must be append-only and tamper-evident** `[COMPLIANCE]` `[OWASP-A08]` `[OWASP-A09]`
  > A log that can be modified after the fact provides no evidentiary value. Use append-only storage or cryptographic chaining.

- [ ] **Log all access to sensitive data who, what, when** `[COMPLIANCE]` `[OWASP-A09]`
  > Required for data subject access requests under most data protection regimes. Also your first signal when something goes wrong.

- [ ] **Alert on anomalous AI usage patterns** `[AI THREAT]` `[OWASP-A09]` `[ATLAS-T0006]` `[NIST-MS]`
  > Unusual query volumes, unexpected output lengths, or repeated injection-pattern inputs should trigger alerts, not just log entries.

- [ ] **If the AI takes actions, log every action with the five facts** `[CRITICAL]` `[ASI10]` `[OWASP-A09]`
  > Who acted, which tool, with what arguments, under which credential, and what triggered it. Prompt logs alone can't tell you what an agent did. See `core-ai/observability.md` §4.

---

## 8. Compliance & Governance Baseline

> Frameworks: `[LLM09]` `[OWASP-A04]` `[NIST-GV]` `[NIST-MP]`

- [ ] **Disclose AI use to end users** `[CRITICAL]` `[COMPLIANCE]` `[LLM09]` `[NIST-GV]`
  > Users should know when they are interacting with or receiving output from an AI system. This is a regulatory requirement in an increasing number of jurisdictions.

- [ ] **Document your data flows what data, where stored, who accesses it, how long retained** `[COMPLIANCE]` `[NIST-MP]`
  > The foundation of any data protection compliance posture. Without this you cannot respond to data subject requests or demonstrate compliance.

- [ ] **Explicit, informed consent before collecting user data** `[CRITICAL]` `[COMPLIANCE]` `[NIST-GV]`
  > Consent must be specific, freely given, and not bundled with unrelated terms. Pre-ticked boxes do not count.

- [ ] **Write an incident response runbook before you have users** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A04]` `[NIST-MG]`
  > Most data protection regimes require breach notification within 72 hours. You need a runbook before the clock starts, not after.

- [ ] **Vendor risk assessment for all AI providers and data processors** `[COMPLIANCE]` `[LLM05]` `[NIST-GV]`
  > If your AI provider processes user data, they are a data processor. You are responsible for assessing and documenting their security posture.
