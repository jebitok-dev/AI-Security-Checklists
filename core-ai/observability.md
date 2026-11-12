# Observability: Logging, Audit Trails & Monitoring

A consolidated lens on logging for systems with an AI layer. Pulls every observability-relevant item from the rest of the repo into one place so a team auditing "do we log enough?" has one file to read.

Items here are not new requirements they cross-reference back to their home in the domain checklists. The value of this file is that the answer to "what should we log?" lives in a single place instead of being scattered across five.

Logging is the most commonly skipped control. Teams ship features and add logging "later" and later never comes until something goes wrong. OWASP A09 (Security Logging and Monitoring Failures) exists exactly because of this. Retrofitting observability into a live system is significantly harder than building it in.

---

## Table of Contents

1. [What to log](#1-what-to-log)
2. [Properties every log must have](#2-properties-every-log-must-have)
3. [AI-specific logging](#3-ai-specific-logging)
4. [Agent action logging](#4-agent-action-logging)
5. [Dependency & supply chain observability](#5-dependency--supply-chain-observability)
6. [Access, override & audit logging](#6-access-override--audit-logging)
7. [Retention, access & destruction](#7-retention-access--destruction)
8. [Alerting & monitoring](#8-alerting--monitoring)
9. [What NOT to log](#9-what-not-to-log)
10. [Verifying your logging works](#10-verifying-your-logging-works)

---

## 1. What to log

> Frameworks: `[OWASP-A09]` `[NIST-MS]`

The minimum event set every production system should emit.

- [ ] **Every authentication event login, logout, failed attempt, password reset, MFA challenge** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A09]`
  > Cross-ref: `web-app-ai/checklist.md` §11. Without auth logs you cannot investigate account compromise or demonstrate compliance with most data protection regimes.

- [ ] **Every authorisation decision allow and deny, with the resource and the requesting identity** `[OWASP-A09]` `[OWASP-A01]`
  > Deny logs are where lateral movement and privilege escalation attempts show up. Drop them and you lose your strongest detection signal.

- [ ] **Every AI prompt and the corresponding output** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A09]` `[LLM06]`
  > Cross-ref: `core-ai/ai-guardrails.md` §7. The minimum viable AI audit trail. Needed for debugging, compliance, and repudiation defence.

- [ ] **Every access to sensitive data who, what, when, why** `[COMPLIANCE]` `[OWASP-A09]`
  > Cross-ref: `core-ai/ai-guardrails.md` §7. Required for data subject access requests in most data protection regimes. Also your first signal when something goes wrong.

- [ ] **Every administrative action role changes, config changes, deletions** `[CRITICAL]` `[OWASP-A09]`
  > Admin actions are the highest-impact events in your system. Anything that changes access control, configuration, or persistent state must be logged.

- [ ] **Every payment, financial, or clinical decision-related event** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A09]`
  > Cross-ref: `fintech-ai/checklist.md` §1, `health-ai/checklist.md` §2. These are the events regulators will ask to see in any incident.

---

## 2. Properties every log must have

> Frameworks: `[OWASP-A09]` `[OWASP-A08]` `[NIST-MS]`

Logs that lack these properties are not evidence. They are story-telling.

- [ ] **Timestamps with timezone (UTC preferred) and millisecond precision where feasible** `[CRITICAL]` `[OWASP-A09]`
  > Cross-system correlation is impossible without consistent timestamps. Pick one zone, ideally UTC, and enforce it everywhere.

- [ ] **User ID and session ID on every log line where applicable** `[CRITICAL]` `[OWASP-A09]`
  > "Something went wrong" is not actionable. "User 12345 in session abc-def hit error X at 14:32:01 UTC" is.

- [ ] **Append-only storage logs cannot be modified after write** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A08]` `[OWASP-A09]`
  > Cross-ref: `core-ai/ai-guardrails.md` §7. A log that can be modified after the fact has no evidentiary value. Use append-only storage (CloudWatch, immutable object storage, ledger DB) or cryptographic chaining.

- [ ] **Tamper-evident chain or signature** `[COMPLIANCE]` `[OWASP-A08]`
  > For high-value or regulated systems, append-only is not enough you need to detect tampering attempts. Hash-chained logs or signed log shipments.

- [ ] **Structured (JSON or similar), not free-text** `[OWASP-A09]`
  > Free-text logs can't be queried, alerted on, or correlated reliably. Structured logging from day one is significantly cheaper than retrofitting.

- [ ] **Correlation ID propagated across services** `[OWASP-A09]`
  > A single user action can touch five services. Without a shared correlation ID, reconstructing the chain is forensically expensive.

---

## 3. AI-specific logging

> Frameworks: `[LLM05]` `[LLM06]` `[LLM09]` `[OWASP-A08]` `[OWASP-A09]` `[ATLAS-T0006]`

Beyond standard application logging, AI systems have additional events worth capturing.

- [ ] **Model name and version on every AI output** `[CRITICAL]` `[COMPLIANCE]` `[LLM05]` `[OWASP-A08]`
  > Cross-ref: `core-ai/ai-guardrails.md` §3, `fintech-ai/checklist.md` §1. If your AI model changes and outputs shift, you need to know which model produced which output. If an output is disputed months later, this is non-negotiable.

- [ ] **System prompt version (hash or identifier) on every AI output** `[COMPLIANCE]` `[LLM05]` `[OWASP-A08]`
  > System prompts evolve. The output a user saw on March 1 may not be reproducible on May 1 unless you logged the prompt version.

- [ ] **Retrieval corpus version on every RAG-assisted output** `[COMPLIANCE]` `[LLM05]` `[OWASP-A08]`
  > A RAG store grows and changes. The documents that informed an output last week may have been deleted or replaced.

- [ ] **Data provenance for inputs that came from third-party sources** `[AI THREAT]` `[COMPLIANCE]` `[LLM03]` `[OWASP-A08]`
  > Cross-ref: `fintech-ai/checklist.md` §5. "This figure came from Vendor X at timestamp T" is a materially different input than untagged data. Especially important when third-party data is later disputed or proven wrong.

- [ ] **Immutable record of AI input and output, frozen at presentation time** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A08]`
  > Cross-ref: `core-ai/ai-guardrails.md` §4, `fintech-ai/checklist.md` §1, `health-ai/checklist.md` §2. The exact output a decision-maker saw must be preserved as-is. Retroactive modification destroys the evidentiary chain.

- [ ] **Tool/function call traces for agentic AI which tool, what arguments, what result** `[CRITICAL]` `[LLM07]` `[LLM08]` `[ASI02]` `[OWASP-A09]`
  > Agentic AI takes actions. Logging the chain of tool calls is the only way to reconstruct what an agent did and why. See §4 for the full set of agent logging requirements.

- [ ] **AI confidence/uncertainty score on every output** `[COMPLIANCE]` `[LLM09]`
  > Cross-ref: `health-ai/checklist.md` §2. If the AI expressed low confidence and a decision-maker proceeded anyway, that is itself evidence both for them and against them.

---

## 4. Agent action logging

> Frameworks: `[ASI02]` `[ASI03]` `[ASI06]` `[ASI07]` `[ASI08]` `[ASI09]` `[ASI10]` `[OWASP-A09]` `[LLM08]`
> Applies when the AI can call tools, keep memory, run code, or delegate to other agents. See `frameworks/owasp-agentic-top-10.md`.

Prompt-and-output logging (§3) tells you what the AI *said*. Once it can act, you also need to know what it *did*, under whose authority, and what made it do it. The OWASP agentic guidance is explicit that this logging is the deployer's job: the model provider does not do it for you.

- [ ] **Every agent action records the five facts** `[CRITICAL]` `[ASI10]` `[OWASP-A09]`
  > 1. **Who acted:** agent ID, and the user it acted for.
  > 2. **Which tool:** tool name and version.
  > 3. **With what details:** the arguments, or a hash of them where they contain sensitive data.
  > 4. **Which credential:** the identity the call ran under (`identity=`).
  > 5. **What started it:** user message ID, scheduled job, upstream agent, or the document/tool result that led to it.
  >
  > Plus timestamp, session ID, and a payload hash so the record can be checked against what actually ran. Cross-ref: `web-app-ai/checklist.md` §12, `fintech-ai/checklist.md` §2.

- [ ] **The credential used is logged separately from the agent and the user** `[CRITICAL]` `[ASI03]` `[OWASP-A09]`
  > "Agent X, acting for user Y, using service account Z." If those three collapse into one field, you cannot spot an agent using broader rights than the user who asked.

- [ ] **Trace IDs carried across agent handovers and inter-agent messages** `[CRITICAL]` `[ASI07]` `[ASI08]` `[OWASP-A09]`
  > Extends the correlation ID in §2 to agents. Log sender, receiver, and whether authentication or signature checks passed. A rejected message is a security event, not noise.

- [ ] **Approvals logged with who approved, when, what they were shown, and the exact arguments approved** `[CRITICAL]` `[COMPLIANCE]` `[ASI09]` `[OWASP-A09]`
  > Alert if the arguments executed differ from the arguments approved. Cross-ref: `fintech-ai/checklist.md` §2, `health-ai/checklist.md` §2.

- [ ] **Memory writes and deletes logged with source and trust level** `[AI THREAT]` `[ASI06]` `[OWASP-A09]`
  > Record whether the memory came from the user directly or from content the agent read, which session wrote it, and when it expires. Without this, a poisoned memory cannot be traced back to where it came from.

- [ ] **Tool and MCP server version (or content hash) logged on each call** `[ASI04]` `[LLM05]` `[OWASP-A09]`
  > Alert when a tool's version, hash, or description changes. A tool description is prompt input, so a silent change to it is a behaviour change.

- [ ] **Code executions logged with code hash, sandbox ID, and blocked actions** `[ASI05]` `[OWASP-A09]`
  > Blocked network or filesystem attempts from the sandbox are a strong signal of injection. Log them and alert.

- [ ] **Kill-switch use and credential revocations logged** `[ASI10]` `[OWASP-A09]` `[NIST-MG]`
  > Who stopped which agent, when, why, and which credentials were revoked. This is the start of every incident timeline.

- [ ] **Agent logs are stored in your own log pipeline, not only in the model or agent platform's console** `[CRITICAL]` `[OWASP-A09]` `[COMPLIANCE]`
  > Vendor dashboards have their own retention, access, and export limits, and may not record your tool arguments or credentials at all. Ship agent events to the same append-only, access-controlled store as the rest of your audit logs (§2).

---

## 5. Dependency & supply chain observability

> Frameworks: `[OWASP-A06]` `[OWASP-A09]` `[LLM05]` `[ATLAS-T0010]`

CI-time CVE scanning catches what was known when you shipped. A CVE disclosed against your dependencies *next week* is an observability problem this section is about catching that.

- [ ] **Continuous CVE monitoring against the dependency set in production** `[CRITICAL]` `[OWASP-A06]` `[LLM05]`
  > Cross-ref: `web-app-ai/checklist.md` §10. Pre-merge CVE scans catch known CVEs at build time. They do not catch CVEs disclosed *after* you ship. Production needs continuous monitoring: Dependabot, Snyk, OSV-Scanner, GitHub Advanced Security, or equivalent, running on a schedule against the deployed manifest.

- [ ] **Python: pip-audit or Safety run on a schedule against the deployed lockfile** `[OWASP-A06]`
  > `pip-audit` queries the PyPI Advisory Database and OSV.dev. `Safety` queries its own DB. Run nightly or weekly against the actual production lockfile (`requirements.txt`, `poetry.lock`, `Pipfile.lock`), not the dev environment.

- [ ] **JavaScript/Node: npm audit and OSV-Scanner on schedule** `[OWASP-A06]`
  > Equivalent coverage for Node services. Lockfile is the source of truth `package-lock.json` or `yarn.lock`.

- [ ] **SBOM logged at every deploy what versions are running right now?** `[COMPLIANCE]` `[OWASP-A06]` `[OWASP-A09]`
  > When a CVE is disclosed, the first question is "are we affected and since when?". The answer requires a per-deploy SBOM snapshot tied to a timestamp. Cross-ref: `web-app-ai/checklist.md` §10.

- [ ] **Alert when a new CVE is published against a dependency currently in production** `[CRITICAL]` `[OWASP-A09]`
  > Routing this alert to a humans-watch channel is what closes the gap between disclosure and patch. The default Dependabot alert lands in the repo's Security tab people miss those. Mirror them into the on-call alerting channel.

- [ ] **Alert on suspicious dependency behaviour version yanked, package transferred, maintainer changed** `[AI THREAT]` `[LLM05]` `[ATLAS-T0010]`
  > Supply chain attacks usually start with a maintainer takeover or a quietly-published malicious version. Tools like Socket, Phylum, or Snyk flag these. For Python: monitor PyPI release activity for your direct dependencies.

- [ ] **Model artifact integrity verified at load time** `[CRITICAL]` `[AI THREAT]` `[LLM05]` `[ATLAS-T0010]`
  > Pre-trained models pulled from Hugging Face or similar are dependencies too. Verify a known-good hash at deploy time and log any mismatch as a P1 event. Cross-ref: `core-ai/ai-guardrails.md` §3.

- [ ] **Time-to-patch tracked as a metric** `[NIST-MS]`
  > How long between a critical CVE being published and the patched version reaching production? If you don't measure it, it gets worse. Aim for under 7 days for critical, 30 for high.

---

## 6. Access, override & audit logging

> Frameworks: `[OWASP-A09]` `[OWASP-A01]` `[NIST-MG]`

Not just *what* happened, but *who* did it and *who saw it*.

- [ ] **Break-glass / emergency access events flagged immediately** `[CRITICAL]` `[COMPLIANCE]` `[OWASP-A09]` `[NIST-MG]`
  > Cross-ref: `health-ai/checklist.md` §5. Emergency access bypasses normal controls. Log it, notify automatically, and review within 24 hours.

- [ ] **Override logging when a human overrides an AI recommendation** `[COMPLIANCE]` `[AI THREAT]` `[OWASP-A09]` `[NIST-MS]`
  > Cross-ref: `health-ai/checklist.md` §2. Log what the AI recommended, what the human decided, and ideally why. This is both an audit trail and a quality improvement signal.

- [ ] **Data subject access events who read whose data, when** `[COMPLIANCE]` `[OWASP-A09]`
  > Required to satisfy data subject access requests. "Who has read my record?" is a question every data protection regime expects you to be able to answer.

- [ ] **Audit log read access who reads the audit log itself** `[COMPLIANCE]` `[OWASP-A09]`
  > The audit log is the most sensitive log. Log who reads it. This is meta but critical: if an attacker compromises your monitoring, they will read the logs first.

- [ ] **Failed authorisation attempts at sensitive resources** `[OWASP-A09]` `[OWASP-A01]`
  > A user repeatedly hitting endpoints they shouldn't access is a strong reconnaissance signal. Bury this in normal traffic logs and you'll miss it.

---

## 7. Retention, access & destruction

> Frameworks: `[COMPLIANCE]` `[NIST-GV]`

Logs are personal data. They have a lifecycle.

- [ ] **Log retention policy defined per log category** `[COMPLIANCE]`
  > Cross-ref: `web-app-ai/checklist.md` §11. Different data has different retention requirements. Auth logs, AI prompts, clinical data, payment data each needs its own policy.

- [ ] **Retention enforced technically, not just documented** `[COMPLIANCE]`
  > A retention policy in a Notion page is not enforcement. Logs auto-expire via TTL, lifecycle rules, or scheduled deletion.

- [ ] **Logs containing patient/clinical data subject to clinical-record retention** `[COMPLIANCE]`
  > Cross-ref: `health-ai/checklist.md` §3. If the AI interaction is part of the clinical record, it inherits the clinical retention rules (often 7–10 years).

- [ ] **Logs containing financial data subject to financial-record retention** `[COMPLIANCE]`
  > Cross-ref: `fintech-ai/checklist.md` §1. Audit trails for financial decisions have jurisdiction-specific minimums.

- [ ] **Access to logs is itself permissioned and logged** `[OWASP-A09]` `[OWASP-A01]`
  > Engineers should not have blanket access to production logs containing user data. Role-based access, just-in-time approval, with reads logged.

- [ ] **Logs included in data subject deletion (right to be forgotten) procedures** `[COMPLIANCE]`
  > If a user exercises a deletion right, you must be able to remove their data from logs too or be able to justify why a specific log category is exempt (e.g. legal hold).

---

## 8. Alerting & monitoring

> Frameworks: `[OWASP-A09]` `[NIST-MS]` `[NIST-MG]` `[ATLAS-T0006]`

A log that no one reads is not monitoring. Alerts are the contract between logs and humans.

- [ ] **Define thresholds; route alerts to a channel a human watches** `[CRITICAL]` `[OWASP-A09]`
  > Cross-ref: `web-app-ai/checklist.md` §11. Monitoring without alerting is theatre. Each alert needs a documented owner and a runbook.

- [ ] **Alert on auth anomalies repeated failures, lockouts, new device, impossible travel** `[OWASP-A09]` `[OWASP-A07]`
  > These are the classic account-compromise signals. Catching them early is the difference between a near-miss and a breach.

- [ ] **Alert on anomalous AI usage volume, output length, suspected injection patterns** `[AI THREAT]` `[LLM01]` `[ATLAS-T0006]`
  > Cross-ref: `core-ai/ai-guardrails.md` §7. Unusual query volumes, unexpected output lengths, or repeated injection-pattern inputs are the AI-layer equivalents of brute-force or scan traffic.

- [ ] **Alert on cost-harvesting signals high token consumption per user, unusual model selection** `[AI THREAT]` `[LLM04]` `[ATLAS-T0034]`
  > Financial DoS via AI is real and underestimated. Per-user cost monitoring is now an operational requirement, not a finance-team nicety.

- [ ] **Alert on model drift fraud detection rates shifting, quality scores changing** `[AI THREAT]` `[LLM03]`
  > Cross-ref: `fintech-ai/checklist.md` §3, `health-ai/checklist.md` §4. Sudden shifts indicate drift, poisoning, or a discovered evasion technique.

- [ ] **Alert when an agent leaves its behavioural baseline new tools, new credentials, new recipients, unusual volume** `[CRITICAL]` `[AI THREAT]` `[ASI10]`
  > Declare each agent's expected tools and goals, record what normal looks like, and alert on deviation. This is how a hijacked or drifting agent gets caught while it is still acting.

- [ ] **Alert on agent actions with no resolvable trigger ("origin unknown")** `[AI THREAT]` `[ASI10]` `[STRIDE]`
  > Every action should trace to a user, a schedule, or another agent. One that traces to nothing is either a logging gap or an intrusion; both need a human.

- [ ] **Alert when an agent hits a tool-call, loop, or spend cap** `[ASI08]` `[LLM04]`
  > Cross-ref: `web-app-ai/checklist.md` §12. Hitting a cap means the cap worked, but something upstream is wrong.

- [ ] **Suppress alert fatigue tune thresholds, deduplicate, escalate properly** `[OWASP-A09]`
  > An alert channel everyone mutes is worse than no alerting at all. Tuning is ongoing work, not a one-time setup.

- [ ] **Incident response runbook for each alert category** `[CRITICAL]` `[COMPLIANCE]` `[NIST-MG]`
  > Cross-ref: `web-app-ai/checklist.md` §11, `core-ai/ai-guardrails.md` §8. Who is paged? What do they check first? What is the escalation path? Document before the page fires.

---

## 9. What NOT to log

> Frameworks: `[OWASP-A02]` `[LLM06]` `[COMPLIANCE]`

Logging the wrong thing turns your audit trail into a breach.

- [ ] **No raw passwords, password reset tokens, or session tokens** `[CRITICAL]` `[OWASP-A02]`
  > Tokens in logs are credentials in logs. Anyone with log access has those credentials.

- [ ] **No raw payment card data PANs, CVVs, expiry** `[CRITICAL]` `[OWASP-A02]` `[COMPLIANCE]`
  > Cross-ref: `fintech-ai/checklist.md` §2. PCI DSS violation. Tokenize or mask before logging.

- [ ] **No raw API keys or secrets in error messages** `[CRITICAL]` `[OWASP-A02]`
  > Stack traces that include the secret being used are a common leak vector. Scrub at the logger layer, not at the call site.

- [ ] **No agent credentials, tool tokens, or connection strings in agent logs mask or hash them** `[CRITICAL]` `[OWASP-A02]` `[ASI03]`
  > Tool arguments and tool results often carry tokens and connection strings. Log a hash or a masked reference (`identity=svc-payments-agent`), never the secret itself. Hashing still lets you match a record to what ran.

- [ ] **No full prompts or outputs containing sensitive data unless required and protected** `[LLM06]` `[COMPLIANCE]`
  > AI prompts often contain user-provided sensitive data. If you log them, they become an additional data store with the same protection requirements as the original.

- [ ] **No PII in URLs that hit access logs** `[OWASP-A02]`
  > Cross-ref: `web-app-ai/checklist.md` §9. Query strings end up in web server logs, CDN logs, browser history, and Referer headers. Use POST body or headers.

- [ ] **No raw user content where minimisation suffices** `[COMPLIANCE]` `[LLM06]`
  > If you only need to know "user submitted a query of length 4032", logging the full query is over-collection. Default to minimum, escalate to full only when a specific reason exists.

---

## 10. Verifying your logging works

> Frameworks: `[NIST-MS]`

Logging that has never been tested often turns out to be broken.

- [ ] **Run a synthetic incident does your logging answer who-what-when?** `[CRITICAL]` `[OWASP-A09]`
  > Pick a recent real incident shape auth compromise, prompt injection, data leak. Walk through the logs as if responding. Note where you go blind.

- [ ] **Run a synthetic agent incident can you trace an action back to what triggered it?** `[CRITICAL]` `[ASI01]` `[ASI10]` `[OWASP-A09]`
  > Plant an instruction in a document, let the agent read it and take a (harmless) action, then work backwards from the action using only the logs. You should reach the document, the user, the agent, and the credential. Where you can't, that's the gap.

- [ ] **Verify log volume against expected** `[OWASP-A09]`
  > Logging silently dropping events is a common failure mode. If you expect 10k auth events/day and see 100, something is broken.

- [ ] **Verify alerts fire end-to-end** `[OWASP-A09]`
  > Trigger the conditions for each alert in a test environment. Confirm it reaches the on-call channel. Tabletop is not enough.

- [ ] **Verify retention deletion actually deletes** `[COMPLIANCE]`
  > Check that data past its retention window is gone from primary storage *and* backups *and* search indices *and* archive storage.

- [ ] **Verify tamper-evidence run a chaos test where someone tries to modify a log** `[OWASP-A08]`
  > If you cannot detect a modified log, your tamper-evidence is theatre.

---

## Cross-references

This file consolidates items from:

| Source | Sections |
|--------|----------|
| `core-ai/ai-guardrails.md` | §3 (model provenance, supply chain), §4 (immutable audit log), §7 (full logging section), §8 (incident runbook) |
| `web-app-ai/checklist.md` | §10 (dependency CVE scan, SBOM — extended here into continuous monitoring), §11 (logging, monitoring, IR), §12 (agent action logging, approvals, caps) |
| `fintech-ai/checklist.md` | §1 (audit trail for AI-assisted decisions, model version), §2 (agent-initiated payment logging) |
| `health-ai/checklist.md` | §2 (override logging, AI recommendations auditable, AI write logging), §3 (transcript retention), §5 (break-glass) |
| `frameworks/owasp-agentic-top-10.md` | Logging note (five facts per action), ASI03, ASI07–ASI10 |

If you change an item here, change it in the source file too or restructure so this file becomes the source of truth and the others reference back.

---

## References

- [OWASP A09: Security Logging and Monitoring Failures](https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/)
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/)
- [NIST AI RMF: Measure function](https://www.nist.gov/itl/ai-risk-management-framework)
- [MITRE ATLAS: Active Scanning (AML.T0006)](https://atlas.mitre.org/techniques/AML.T0006)
- [PortSwigger: Logging vulnerabilities](https://portswigger.net/web-security/logging)
- [pip-audit](https://pypi.org/project/pip-audit/) — Python dependency CVE scanner
- [PyPI Advisory Database](https://github.com/pypa/advisory-database) — source for Python CVEs
- [OSV.dev](https://osv.dev/) — cross-ecosystem vulnerability database
- [GitHub Dependabot](https://docs.github.com/en/code-security/dependabot)
