# AI Security Checklists

Practitioner checklists for securing web applications with AI integration.

Most security checklists stop at the web layer. These go further mapping traditional vulnerability classes to the new attack surfaces that appear when you add an AI layer to a real system with real users and real consequences.

Built from hands-on CTF work, OWASP LLM Top 10, MITRE ATLAS, STRIDE threat modelling, and real engagement experience.

---

## Table of Contents

- [Structure](#structure)
- [How to use this](#how-to-use-this)
- [Tags](#tags)
- [Frameworks referenced](#frameworks-referenced)
- [Contributing](#contributing)
- [Author](#author)

---

## Structure

```
core-ai/
  ai-guardrails.md            # AI-specific checks shared across all domains
  observability.md            # Consolidated lens on logging, audit trails, monitoring
  prompt-injection-payloads.md # Categorised payload library for testing
  known-incidents.md          # Public AI incidents, root causes, prevention
web-app-ai/
  checklist.md            # Full checklist for web apps with AI integration
  attack-scenarios.md     # Real attack scenarios per vulnerability class
fintech-ai/
  checklist.md            # Fintech-specific additions
health-ai/
  checklist.md            # Health/medical-specific additions
frameworks/
  owasp-llm-top-10.md     # LLM01-LLM10 reference, mapped to checklist items
  owasp-top-10-2025.md    # OWASP Top 10 (web) 2025 reference
  mitre-atlas-top.md      # Most relevant MITRE ATLAS techniques for web+AI
  owasp-agentic-top-10.md # ASI01-ASI10 reference for agentic systems (tools, memory, delegation)
```

---

## How to use this

- Start with `core-ai/ai-guardrails.md` these apply to any system with an AI layer
- Then pick your domain checklist
- If your AI can call tools, persist memory, or delegate to other agents, also work through `frameworks/owasp-agentic-top-10.md`
- Items tagged `[CRITICAL]` should be addressed before any users touch the system
- Items tagged `[AI THREAT]` are specific to the AI layer standard security tools won't catch these
- Items tagged `[COMPLIANCE]` have regulatory implications depending on your jurisdiction

---

## Tags

| Tag | Meaning |
|-----|---------|
| `[CRITICAL]` | Must be addressed before launch or beta |
| `[AI THREAT]` | AI-layer specific: prompt injection, model integrity, data poisoning |
| `[STRIDE]` | Maps to a STRIDE threat category |
| `[COMPLIANCE]` | Has regulatory / legal implications |
| `[LLM0X]` | Maps to an OWASP LLM Top 10 category (LLM01–LLM10) |
| `[OWASP-A0X]` | Maps to an OWASP Top 10 (web) 2025 category (A01–A10) |
| `[ASI0X]` | Maps to an OWASP Top 10 for Agentic Applications category (ASI01–ASI10) |
| `[ATLAS-Txxxx]` | Maps to a MITRE ATLAS technique ID |
| `[NIST-XX]` | Maps to a NIST AI RMF function (GV / MP / MS / MG) |

See `frameworks/` for the full reference of each tagged framework.

---

## Frameworks referenced

- OWASP LLM Top 10 (see `frameworks/owasp-llm-top-10.md`)
- OWASP Top 10 for Web Applications 2025 (see `frameworks/owasp-top-10-2025.md`)
- OWASP Top 10 for Agentic Applications 2026 (see `frameworks/owasp-agentic-top-10.md`)
- MITRE ATLAS (see `frameworks/mitre-atlas-top.md`)
- OWASP Web Security Testing Guide (WSTG)
- MITRE ATT&CK
- STRIDE threat modelling
- NIST AI Risk Management Framework (AI RMF 1.0)

---

## Contributing

PRs welcome. If you find a gap, a new attack pattern, or a domain-specific addition open an issue or submit a checklist item with a one-line rationale.

---

## Author

Built by a security practitioner working across web exploitation, AI security, and CTF research.

- Blog: [jebitok.hashnode.dev](https://jebitok.hashnode.dev)
- GitHub: [@jebitok-dev](https://github.com/jebitok-dev)
