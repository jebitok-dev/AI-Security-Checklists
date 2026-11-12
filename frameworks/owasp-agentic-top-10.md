# OWASP Top 10 for Agentic Applications (2026)

Reference mapping for the **agentic layer** — the risks that apply once a model
*acts*: calls tools, holds memory, uses credentials, or delegates to other agents.
Published by the OWASP GenAI Security Project (Dec 2025), codes **ASI01–ASI10**.

This file is the agentic counterpart to [`owasp-llm-top-10.md`](./owasp-llm-top-10.md)
and [`owasp-top-10-2025.md`](./owasp-top-10-2025.md). Use all three together.

> **Scope note — not every AI system is agentic.**
> A system that only *responds* (a RAG app that retrieves and answers) lives at the
> **web** (`[OWASP-A0X]`) and **LLM** (`[LLM0X]`) layers. The **ASI** layer applies
> only once the model can call tools, persist memory, or delegate. Apply this
> checklist to the agentic surface, not to a plain retrieval app. Over-applying it
> is as much an error as skipping web security on an "AI" app.

## New tag

- `[ASI0X]`: maps to an OWASP Top 10 for Agentic Applications category (ASI01–ASI10).
  Threads alongside `[LLM0X]` and `[OWASP-A0X]` — one item can carry all three.

## The ten, at a glance

| Code | Risk | What goes wrong | Primary control |
| --- | --- | --- | --- |
| ASI01 | Agent Goal Hijack | Content the agent reads changes what it's trying to do | Treat all input as untrusted; approval for goal-changing actions |
| ASI02 | Tool Misuse & Exploitation | An allowed tool is used unsafely | Least-privilege tool profiles; policy check before each tool call |
| ASI03 | Identity & Privilege Abuse | Agent borrows/inherits rights it shouldn't hold | Per-agent identity; scoped, short-lived credentials |
| ASI04 | Agentic Supply Chain | A tool, MCP server, prompt, or peer agent is tampered with | Allowlist + hash-pin components; verify provenance; kill switch |
| ASI05 | Unexpected Code Execution (RCE) | Generated/injected code runs on a host | Sandbox without root; ban `eval` in prod; approval for elevated runs |
| ASI06 | Memory & Context Poisoning | Stored memory or retrieved context is corrupted | Scan writes; segment by tenant; record provenance; expire unverified |
| ASI07 | Insecure Inter-Agent Communication | Messages between agents spoofed/replayed/altered | Mutual auth; signed messages with nonces + timestamps; signed agent cards |
| ASI08 | Cascading Failures | One fault spreads across agents and workflows | Isolate agents; circuit breakers; independent policy checks; trace everything |
| ASI09 | Human-Agent Trust Exploitation | A person approves a bad action because the agent sounded confident | Plain-language risk summaries at approval, not model-written rationales |
| ASI10 | Rogue Agents | An agent drifts from intended behaviour and keeps acting | Declare expected tools/goals; baseline behaviour; kill switch + credential revocation |

---

## ASI01 — Agent Goal Hijack

Attacker-controlled content (a web page, a document, a tool result, a retrieved
chunk) silently rewrites the agent's objective or decision path.

- **Related:** `[STRIDE]` Tampering · `[LLM0X]` LLM01 (Prompt Injection) · `[ATLAS-Txxxx]` AML.T0051
- **Threads into:** [`web-app-ai/attack-scenarios.md`](../web-app-ai/attack-scenarios.md) (Scenarios 1–3, 16), [`core-ai/prompt-injection-payloads.md`](../core-ai/prompt-injection-payloads.md), [`fintech-ai/checklist.md`](../fintech-ai/checklist.md) (Scenario 5)

Checklist items:
- [ ] All agent-reachable content (RAG corpus, tool output, web fetches) is treated as untrusted input, not trusted context `[ASI01]` `[LLM01]` `[AI THREAT]`
- [ ] Goal- or plan-changing actions require explicit approval, not just model reasoning `[ASI01]` `[CRITICAL]`
- [ ] Indirect injection is tested from every ingestion path, not just the chat box `[ASI01]` `[AI THREAT]`

## ASI02 — Tool Misuse & Exploitation

A capability the agent is *allowed* to use is driven to do something unsafe
(dangerous arguments, unintended target, excessive scope).

- **Related:** `[STRIDE]` Elevation of Privilege · `[LLM0X]` LLM08 (Excessive Agency), LLM07 (Insecure Plugin Design)
- **Threads into:** [`core-ai/ai-guardrails.md`](../core-ai/ai-guardrails.md) §5, [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §12, [`web-app-ai/attack-scenarios.md`](../web-app-ai/attack-scenarios.md) (Scenario 16)

Checklist items:
- [ ] Each tool has a least-privilege profile (what it may do, on what targets) `[ASI02]` `[CRITICAL]`
- [ ] A policy check runs before each tool call, not only at session start `[ASI02]`
- [ ] Tool arguments are validated/sanitised the same as any user-influenced input (e.g. SQL, shell, path) `[ASI02]` `[AI THREAT]`

## ASI03 — Identity & Privilege Abuse

The agent acts under more authority than it needs, or borrows another identity's
rights. The core of **Least Agency**.

- **Related:** `[STRIDE]` Spoofing / Elevation of Privilege · `[LLM0X]` LLM08 (Excessive Agency)
- **Threads into:** [`core-ai/ai-guardrails.md`](../core-ai/ai-guardrails.md) §5, [`core-ai/observability.md`](../core-ai/observability.md), [`web-app-ai/attack-scenarios.md`](../web-app-ai/attack-scenarios.md) (Scenario 17), [`fintech-ai/checklist.md`](../fintech-ai/checklist.md) §2

Checklist items:
- [ ] Each agent has its own service identity — no shared credential across a pipeline `[ASI03]` `[CRITICAL]`
- [ ] Credentials are scoped to the exact tools/data the agent needs, and are short-lived `[ASI03]`
- [ ] The identity (`identity=`) a call runs under is distinct from, and logged separately to, the acting agent `[ASI03]` `[STRIDE]`
- [ ] Delegated/inherited access is tracked explicitly; a sub-agent does not silently inherit a parent's scope `[ASI03]`

## ASI04 — Agentic Supply Chain

A tool, MCP server, system prompt, or peer agent is malicious or tampered with.

- **Related:** `[OWASP-A0X]` A06 (Vulnerable & Outdated Components) · `[STRIDE]` Tampering
- **Threads into:** [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §10 and §12, [`core-ai/known-incidents.md`](../core-ai/known-incidents.md)

Checklist items:
- [ ] Tools, MCP servers, and peer agents are allowlisted and pinned (content hash / version) `[ASI04]` `[CRITICAL]`
- [ ] Provenance of prompts, tools, and models is verified before use `[ASI04]`
- [ ] Deployed-but-unused framework features (agent components, workflow builders) are disabled or network-isolated, not assumed out of scope `[ASI04]` `[CRITICAL]`
- [ ] A kill switch can disable a compromised component quickly `[ASI04]`

## ASI05 — Unexpected Code Execution (RCE)

Generated or injected code executes on a host — the agent's runtime, a tool, or a
notebook environment.

- **Related:** `[OWASP-A0X]` A03 (Injection) · `[ATLAS-Txxxx]` · `[STRIDE]` Elevation of Privilege
- **Threads into:** [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §12, [`web-app-ai/attack-scenarios.md`](../web-app-ai/attack-scenarios.md) (Scenario 19)

Checklist items:
- [ ] Agent-generated or agent-invoked code runs in a sandbox without root `[ASI05]` `[CRITICAL]`
- [ ] `eval`-style dynamic execution is banned in production paths `[ASI05]`
- [ ] Elevated or host-level runs require explicit approval `[ASI05]`

## ASI06 — Memory & Context Poisoning

Stored memory or retrieved context is corrupted so it influences later turns or
*other users'* sessions. Persistence breaks the "prompting is ephemeral" assumption.

- **Related:** `[LLM0X]` LLM03 (Training Data Poisoning) · `[STRIDE]` Tampering
- **Threads into:** [`core-ai/ai-guardrails.md`](../core-ai/ai-guardrails.md) §2 and §4, [`core-ai/prompt-injection-payloads.md`](../core-ai/prompt-injection-payloads.md), [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §12, [`web-app-ai/attack-scenarios.md`](../web-app-ai/attack-scenarios.md) (Scenario 18)

Checklist items:
- [ ] Writes to agent memory / vector store are scanned and attributed (provenance) `[ASI06]` `[AI THREAT]`
- [ ] Memory is segmented by tenant/user/service — no cross-tenant retrieval `[ASI06]` `[CRITICAL]`
- [ ] Unverified or low-trust memory entries expire rather than persisting indefinitely `[ASI06]`

## ASI07 — Insecure Inter-Agent Communication

Messages between agents are spoofed, replayed, or altered; a peer agent's reply is
trusted as if it were safe context.

- **Related:** `[STRIDE]` Spoofing / Tampering / Repudiation
- **Threads into:** [`core-ai/observability.md`](../core-ai/observability.md), [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §12, [`web-app-ai/attack-scenarios.md`](../web-app-ai/attack-scenarios.md) (Scenario 20)

Checklist items:
- [ ] Agent-to-agent messages are mutually authenticated `[ASI07]` `[CRITICAL]`
- [ ] Messages are signed with nonces and timestamps to prevent spoofing/replay `[ASI07]`
- [ ] A peer agent's output is treated as untrusted content (re-screened for injection), not trusted input `[ASI07]` `[AI THREAT]`

## ASI08 — Cascading Failures

One fault — a hijack, a bad tool result, an overloaded agent — propagates across
agents and workflows.

- **Related:** `[STRIDE]` Denial of Service · `[NIST-XX]` MG
- **Threads into:** [`core-ai/observability.md`](../core-ai/observability.md), [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §12, [`web-app-ai/attack-scenarios.md`](../web-app-ai/attack-scenarios.md) (Scenario 21), [`fintech-ai/checklist.md`](../fintech-ai/checklist.md) §2

Checklist items:
- [ ] Agents are isolated so one failure doesn't take down the chain `[ASI08]`
- [ ] Circuit breakers / rate limits cap runaway tool invocation or loops `[ASI08]` `[CRITICAL]`
- [ ] Every action is traceable end-to-end, including across handovers `[ASI08]` `[STRIDE]`

## ASI09 — Human-Agent Trust Exploitation

A person approves a harmful action because the agent presented it confidently or
buried the risk in a fluent rationale.

- **Related:** `[STRIDE]` Spoofing · `[COMPLIANCE]` (approval records)
- **Threads into:** [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §12, [`fintech-ai/checklist.md`](../fintech-ai/checklist.md) §1–2, [`health-ai/checklist.md`](../health-ai/checklist.md) §2

Checklist items:
- [ ] Approval prompts show a plain-language risk summary, not a model-generated justification `[ASI09]`
- [ ] High-impact actions require explicit confirmation with the concrete effect stated `[ASI09]` `[CRITICAL]`
- [ ] Approval decisions are logged with who approved and on what request `[ASI09]` `[COMPLIANCE]`

## ASI10 — Rogue Agents

An agent drifts from its intended behaviour — new tools, new recipients, new goals —
and keeps acting.

- **Related:** `[STRIDE]` Repudiation · `[NIST-XX]` MS/MG
- **Threads into:** [`core-ai/observability.md`](../core-ai/observability.md), [`web-app-ai/checklist.md`](../web-app-ai/checklist.md) §12

Checklist items:
- [ ] Each agent's expected tools and goals are declared up front `[ASI10]`
- [ ] A behavioural baseline exists per agent (tools used, credentials used, frequency); deviations alert `[ASI10]` `[CRITICAL]`
- [ ] A kill switch can stop an agent and revoke its credentials mid-incident `[ASI10]` `[CRITICAL]`
- [ ] Actions with no resolvable initiating identity ("origin unknown") are flagged `[ASI10]` `[STRIDE]`

---

## Logging note (feeds [`core-ai/observability.md`](../core-ai/observability.md) §4)

For the agentic layer, every action should record **five facts**: who acted, which
tool, with what details, which credential (`identity=`), and what started it — plus
timestamp, agent ID, session ID, and a payload hash (OWASP). Secrets (tokens,
connection strings) are masked or hashed, never logged. Logging is the deployer's
responsibility, not the model provider's.

## Cross-framework map

LLM codes follow the numbering used in [`owasp-llm-top-10.md`](./owasp-llm-top-10.md), and web codes follow [`owasp-top-10-2025.md`](./owasp-top-10-2025.md), so tags here match the rest of the repo.

| ASI | Closest LLM0X | Closest web A0X | STRIDE |
| --- | --- | --- | --- |
| ASI01 | LLM01 | — | Tampering |
| ASI02 | LLM08, LLM07 | A03 | Elevation of Privilege |
| ASI03 | LLM08 | A01 | Spoofing / EoP |
| ASI04 | LLM05 | A06 | Tampering |
| ASI05 | LLM02 | A03 | Elevation of Privilege |
| ASI06 | LLM03 | — | Tampering |
| ASI07 | — | — | Spoofing / Tampering |
| ASI08 | LLM04 | — | Denial of Service |
| ASI09 | LLM09 | — | Spoofing |
| ASI10 | — | — | Repudiation |

Source: [OWASP GenAI Security Project](https://genai.owasp.org/).
