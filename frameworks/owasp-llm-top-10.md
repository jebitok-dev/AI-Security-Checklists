# OWASP Top 10 for Large Language Model Applications

Reference for the `[LLM0X]` tags used across the checklists. Each entry maps the OWASP LLM category to the checklist items that address it, so you can demonstrate coverage in an audit or gap analysis without re-reading every file.

Source: [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

---

## Table of Contents

1. [LLM01: Prompt Injection](#llm01-prompt-injection)
2. [LLM02: Insecure Output Handling](#llm02-insecure-output-handling)
3. [LLM03: Training Data Poisoning](#llm03-training-data-poisoning)
4. [LLM04: Model Denial of Service](#llm04-model-denial-of-service)
5. [LLM05: Supply Chain Vulnerabilities](#llm05-supply-chain-vulnerabilities)
6. [LLM06: Sensitive Information Disclosure](#llm06-sensitive-information-disclosure)
7. [LLM07: Insecure Plugin Design](#llm07-insecure-plugin-design)
8. [LLM08: Excessive Agency](#llm08-excessive-agency)
9. [LLM09: Overreliance](#llm09-overreliance)
10. [LLM10: Model Theft](#llm10-model-theft)
11. [How to use this file in an audit](#how-to-use-this-file-in-an-audit)

---

## LLM01: Prompt Injection

User-controlled input that alters the behaviour of the LLM. Direct (text fields) or indirect (uploaded files, fetched URLs, RAG corpora).

**Mapped checklist items:**
- `core/ai-guardrails.md` §1 (all items)
- `web-app-ai/checklist.md` §4 (web-app-specific prompt injection items)
- `web-app-ai/attack-scenarios.md` Scenarios 1, 2, 3, 13
- `fintech-ai/checklist.md` Scenario 1 (investor-readiness score manipulation)

**Why it matters here:** Any text field, file upload, or fetched URL that reaches the model is an injection point. Standard QA does not test for this.

---

## LLM02: Insecure Output Handling

LLM output rendered to a browser, executed as code, or passed to a downstream system without validation. The classic vector for XSS-via-AI.

**Mapped checklist items:**
- `core/ai-guardrails.md` §6 (Output Handling & Rendering)
- `web-app-ai/checklist.md` §5 (Output Encoding & XSS Prevention)
- `web-app-ai/attack-scenarios.md` Scenario 15 (stored XSS via AI output)

---

## LLM03: Training Data Poisoning

Malicious or biased content injected into training data, fine-tuning data, or RAG retrieval stores. Affects every downstream output.

**Mapped checklist items:**
- `core/ai-guardrails.md` §4 (Data Poisoning & Input Integrity)
- `fintech-ai/checklist.md` §3 (Fraud Detection Model Integrity)
- `health-ai/checklist.md` §4 (AI Model Bias & Fairness)
- `web-app-ai/attack-scenarios.md` Scenario 10 (RAG poisoning)

---

## LLM04: Model Denial of Service

Crafted inputs that exhaust the model's context window, token budget, or API quota. Financial DoS more often than service DoS.

**Mapped checklist items:**
- `core/ai-guardrails.md` §5 (rate limits, spending caps)
- `web-app-ai/checklist.md` §7 (API rate limiting, request size limits)
- `web-app-ai/attack-scenarios.md` Scenarios 11, 12

---

## LLM05: Supply Chain Vulnerabilities

Compromised pre-trained models, vulnerable AI libraries, malicious model hubs. Invisible until triggered.

**Mapped checklist items:**
- `core/ai-guardrails.md` §3 (AI Supply Chain & Model Integrity)
- `web-app-ai/checklist.md` §10 (Dependency & Supply Chain)
- `web-app-ai/attack-scenarios.md` Scenario 9 (compromised dependency)

---

## LLM06: Sensitive Information Disclosure

LLM reveals system prompt content, other users' data, training data, or secrets embedded in the prompt.

**Mapped checklist items:**
- `core/ai-guardrails.md` §2 (AI Context & Tenant Isolation)
- `core/ai-guardrails.md` §3 (no secrets in prompts)
- `web-app-ai/checklist.md` §8 (AI Context Isolation)
- `web-app-ai/attack-scenarios.md` Scenarios 4, 13, 14
- `health-ai/checklist.md` §3 (Sensitive Data Minimisation in AI Prompts)

---

## LLM07: Insecure Plugin Design

LLM plugins, tools, or function-calling integrations with insufficient input validation or excessive permissions. Plugins become a privilege-escalation pathway.

**Mapped checklist items:**
- `core/ai-guardrails.md` §5 (Agentic AI & Permission Controls)
- `web-app-ai/checklist.md` §8 (AI cannot access data stores directly)

---

## LLM08: Excessive Agency

Autonomous LLM agents granted more permissions, autonomy, or scope than the task requires. Force multiplier for any prompt injection.

**Mapped checklist items:**
- `core/ai-guardrails.md` §5 (least privilege, human review gate, URL allowlist)
- `web-app-ai/checklist.md` §8 (RAG results filtered by user permissions)
- `fintech-ai/checklist.md` §1 (human review gate before any financial action)
- `health-ai/checklist.md` §2 (decision support only, never autonomous clinical decisions)

---

## LLM09: Overreliance

Users or downstream systems treat LLM outputs as factual without verification. Risk surface is human and organisational as much as technical.

**Mapped checklist items:**
- `core/ai-guardrails.md` §4 (flag AI-generated content as AI-generated)
- `core/ai-guardrails.md` §8 (disclose AI use to end users)
- `fintech-ai/checklist.md` §1 (AI outputs never presented as final decisions)
- `health-ai/checklist.md` §2 (AI confidence/uncertainty explicitly communicated)

---

## LLM10: Model Theft

Unauthorised extraction, copying, or exfiltration of proprietary models and weights. Includes black-box extraction via repeated querying.

**Mapped checklist items:**
- `core/ai-guardrails.md` §3 (model provenance logging)
- `core/ai-guardrails.md` §7 (alert on anomalous query patterns)
- `web-app-ai/checklist.md` §7 (rate limit all API endpoints)

---

## How to use this file in an audit

1. Pick the LLM category an auditor or stakeholder asks about.
2. Open the mapped checklist sections.
3. Walk through the items and show which are addressed in the current build.
4. For unaddressed items, the rationale or compensating control should be documented alongside.
