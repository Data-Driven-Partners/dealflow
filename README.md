# Dealflow

Research, qualify, build comps, and draft on-voice outreach — every send requires your approval.

[![Claude Cowork](https://img.shields.io/badge/Claude_Cowork-compatible-0F6B5F?style=flat-square)](https://claude.ai/plugins)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-0F6B5F?style=flat-square)](https://docs.anthropic.com/claude/docs/claude-code)
[![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)](LICENSE)
[![Submit to Marketplace](https://img.shields.io/badge/Anthropic_Marketplace-submit-blue?style=flat-square)](https://clau.de/plugin-directory-submission)

---

## What it is

Dealflow is an IB-grade prospecting engine for commercial lenders, SBA brokers, M&A/PE advisors, and agencies. Bring your own contact list (or connect a data source), and the plugin qualifies each prospect against your real fit criteria, researches the company, assembles a comparable-deal comp book, picks a value strategy, and drafts outreach in your configured voice — all queued for your review and approval. Nothing sends until you say so. The engine is fully configurable: pluggable contact sources, four interchangeable value strategies, sector intelligence packs, and A/B split-testing so you learn what actually converts over time.

---

## Getting started

### Claude Cowork

1. Go to [claude.ai/plugins](https://claude.ai/plugins) and search for **Dealflow**.
2. Install `dealflow`.
3. In a Cowork session, run the `setup` skill first: *"Set up my deal-maker prospecting toolkit."*
4. Complete the config/ templates and compliance attestation. Then run `prospect`.

### Claude Code

```bash
claude plugin marketplace add data-driven-partners/dealflow
claude plugin install dealflow
```

After installing, run `setup` before prospecting:

```
Set up my deal-maker prospecting toolkit.
```

Then fill in your `config/` files as prompted. After setup, run `prospect` to start generating drafts.

---

## Core capabilities

| Skill | What it does |
|---|---|
| `setup` | Walks you through filling the config/ templates (company, products, voice, proof sources, compliance attestation). Nothing prospects until setup is complete. |
| `prospect` | The engine: pulls contacts, qualifies against your fit criteria, researches each company, assigns a value strategy by A/B rotation, and queues on-voice drafts for your approval. |
| `deal-voice` | Your configurable voice layer. Reads `config/voice.md` and enforces the facts/beliefs/reframes discipline — no invented claims, no fabricated numbers. |
| `proof-library` | Builds and maintains your proof library: your own closed deals (OURS) and optional public sector data (MARKET). Every proof point is anonymized and provenance-tagged. |

---

## Example prompts

- "Set up my deal-maker prospecting toolkit."
- "Work my dormant list and draft re-engagement outreach."
- "Build a comparable-deals comp book for this prospect."
- "Prospect for HVAC contractors in the Southeast."
- "Refresh my proof library with the latest SBA data."
- "Review this draft for voice — does it sound like us?"

---

## How it works

The `prospect` skill is the orchestration spine. It calls:

1. **A source adapter** — your own contact list (BYO CSV/CRM) or a connected data provider (e.g. DDDB). Each source runs through a hard compliance gate before any contact enters the queue.
2. **A qualification gate** — every contact is evaluated against your fit criteria from `config/products.md`. Contacts that do not fit are excluded with a reason logged. We prospect fits, not lists.
3. **A research sub-agent** — one per surviving prospect (parallel), returning company context, timing signals, and value hooks.
4. **A value strategy** — one of four, assigned by even A/B rotation: `ib-comp-book`, `proof-led`, `timing-led`, `product-reframe`. Each draft is stamped with its strategy, source, sector, and voice version for split-test scoring.
5. **The deal-voice layer** — `config/voice.md` shapes every draft to your configured register and tone.
6. **The approval queue** — every draft is staged for your review. You edit, approve, and send. Nothing goes out automatically.


---

## Data sources and compliance

**BYO list and research are the core, broadly useful capability.** Upload a CSV or connect your CRM and the engine qualifies, researches, and drafts — no external data provider required.

**Connected data providers (e.g. DDDB) are an optional source behind a hard compliance gate.** Activating a connected provider requires completing a five-point compliance attestation in `config/dddb.md` covering: provenance documentation, CAN-SPAM requirements, suppression list management, consent status review, and jurisdiction checks. The engine enforces these checks mechanically — it does not guarantee legal sufficiency in any jurisdiction.

Key guardrails that are always on:

- **Draft-not-send**: every message is staged for your approval. The engine never sends automatically.
- **Suppression**: contacts on your suppression list are excluded unconditionally before each run.
- **No fabrication**: rates, terms, and proof points must trace to `config/products.md` or a cited public source. The engine will not invent numbers.
- **Consent flagging**: contacts with `consent_status = 'none'` are flagged for your review, not auto-queued.

**Cold outreach to purchased or licensed contacts requires your own legal review.** CAN-SPAM, CPRA (California), GDPR (EU), CASL (Canada), and other laws impose obligations on you as the sender. The compliance attestation documents your confirmation of these requirements — it does not substitute for qualified legal counsel.

---

## License

MIT — see [LICENSE](LICENSE).
