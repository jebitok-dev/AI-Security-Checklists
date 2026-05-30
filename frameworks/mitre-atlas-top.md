# MITRE ATLAS - Top Techniques for Web + AI

Reference for the `[ATLAS-Txxxx]` tags used across the checklists. ATLAS (Adversarial Threat Landscape for Artificial-Intelligence Systems) is the AI-focused counterpart to MITRE ATT&CK.

This file covers the techniques most relevant to web applications with an integrated AI layer not the full catalogue. For the complete matrix, see [atlas.mitre.org](https://atlas.mitre.org/).

Format mirrors ATT&CK: techniques are grouped by tactic and tagged with their ATLAS ID.

---

## Table of Contents

- [Reconnaissance](#reconnaissance) - T0000, T0006
- [Initial Access](#initial-access) - T0010, T0049
- [ML Model Access](#ml-model-access) - T0040
- [Execution](#execution) - T0050, T0051, T0053
- [Persistence](#persistence) - T0020
- [Defense Evasion](#defense-evasion) - T0015
- [Collection / Exfiltration](#collection--exfiltration) - T0024, T0035
- [Impact](#impact) - T0034, T0048
- [How to use this file](#how-to-use-this-file)

---

## Reconnaissance

### AML.T0000: Search for Victim's Publicly Available Research Materials

Attacker reads model cards, papers, and blog posts to identify weaknesses in a target's AI stack before any active probe.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §3 (do not publish system prompt internals)

---

### AML.T0006: Active Scanning

Attacker probes the deployed AI system with crafted queries to identify model behaviour, decision boundaries, or system prompt fragments.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §7 (Alert on anomalous AI usage patterns)
- `web-app-ai/checklist.md` §11 (Alerts on threshold breaches)

---

## Initial Access

### AML.T0010: ML Supply Chain Compromise

Attacker compromises a model, dataset, or ML library before it reaches the target the AI-equivalent of a software supply chain attack.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §3 (Treat third-party models as untrusted)
- `web-app-ai/checklist.md` §10 (Dependency CVE scan, SBOM)
- `web-app-ai/attack-scenarios.md` Scenario 9

---

### AML.T0049: Exploit Public-Facing Application

Standard web application exploitation as the entry point into an AI-integrated stack. The AI layer is reached *through* the web application's auth and access control.

**Mapped checklist items:**
- All of `web-app-ai/checklist.md` §1, §2, §3, §7

---

## ML Model Access

### AML.T0040: ML Model Inference API Access

Attacker uses the public inference API as their probe surface: black-box extraction, prompt injection, jailbreaks.

**Mapped checklist items:**
- `web-app-ai/checklist.md` §7 (API rate limiting, authenticated endpoints)
- `core-ai/ai-guardrails.md` §5 (per-user rate limits and spending caps)

---

## Execution

### AML.T0050: Command and Scripting Interpreter

AI agent executes code or shell commands produced by the model. Without sandbox isolation, prompt injection becomes RCE.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §6 (Do not execute AI-generated code without sandbox isolation)
- `core-ai/ai-guardrails.md` §5 (Least privilege for AI components)

---

### AML.T0051: LLM Prompt Injection

Direct or indirect injection of attacker-controlled instructions into the model's input. The single most exploitable AI-layer technique.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §1 (full section)
- `web-app-ai/checklist.md` §4
- `web-app-ai/attack-scenarios.md` Scenarios 1, 2, 3
- `fintech-ai/checklist.md` Scenario 1

---

### AML.T0053: LLM Plugin Compromise

Compromise of a plugin or tool an LLM can invoke. The plugin runs with whatever permissions the agent holds.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §5 (least privilege, URL allowlist, human review gate)

---

## Persistence

### AML.T0020: Poison Training Data

Persistent corruption of the model via training-data manipulation. Survives across deployments until retraining is performed cleanly.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §4 (Data Poisoning & Input Integrity)
- `fintech-ai/checklist.md` §3 (Fraud model training data integrity)
- `web-app-ai/attack-scenarios.md` Scenario 10 (RAG poisoning)

---

## Defense Evasion

### AML.T0015: Evade ML Model

Crafted inputs that cause the model to produce attacker-desired output without triggering detection adversarial examples, jailbreaks, structuring.

**Mapped checklist items:**
- `fintech-ai/checklist.md` §3 (Adversarial testing on fraud model)
- `fintech-ai/checklist.md` Scenario 3 (fraud model evasion)

---

## Collection / Exfiltration

### AML.T0024: Exfiltration via ML Inference API

Attacker extracts training data, system prompts, or other users' data by repeatedly querying the inference API.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §2 (AI Context & Tenant Isolation)
- `core-ai/ai-guardrails.md` §3 (No secrets in prompts)
- `web-app-ai/attack-scenarios.md` Scenarios 3, 4, 13

---

### AML.T0035: ML Artifact Collection

Theft of model weights, configs, or fine-tuned variants.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §3 (Log model provenance)
- `web-app-ai/checklist.md` §2 (Authorisation on every API endpoint)

---

## Impact

### AML.T0034: Cost Harvesting

Attacker drives up the target's AI inference costs financial DoS via expensive queries.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §5 (Hard AI API spending caps, per-user rate limits)
- `web-app-ai/attack-scenarios.md` Scenario 11

---

### AML.T0048: External Harms

Downstream real-world harm caused by manipulated AI output financial loss, clinical harm, reputational damage.

**Mapped checklist items:**
- `core-ai/ai-guardrails.md` §4 (Flag AI-generated content)
- `core-ai/ai-guardrails.md` §5 (Human review gate before irreversible actions)
- `fintech-ai/checklist.md` §1 (AI outputs never presented as final decisions)
- `health-ai/checklist.md` §2 (Decision support only, never autonomous clinical decisions)

---

## How to use this file

ATLAS tactics map roughly to ATT&CK tactics, so a security team already using ATT&CK can adopt this in their existing detection-engineering workflow:

1. For each technique above, identify whether you have detection coverage (logs, alerts, manual review).
2. Where coverage is missing, the linked checklist items name the preventive control.
3. For red team exercises, the technique IDs give a shared vocabulary with the blue team.
