---
name: prospect
description: IB-grade prospecting spine for deal makers. Pulls contacts from DDDB or a BYO list, qualifies against fit criteria, researches, assigns a value strategy by even rotation (ib-comp-book / proof-led / timing-led / product-reframe), and queues on-voice drafts for approval. Every draft is A/B-stamped. Fire when: "prospect", "find leads", "build outreach", "work my list", "generate pipeline", "tee up drafts". Do NOT use when config/ is incomplete or dddb.md attestation is missing — run setup first.
metadata:
  version: 1.0.0
  author: Data Driven Partners
allowed-tools: [WebSearch, Gmail, Supabase, Read, Write, Task]
---

<!-- DDDB pull is via the configured DDDB connector (see config/dddb.md for the endpoint/credential reference).
     Supabase is the buyer's own instance (connection string in environment — never committed).
     Draft-not-send: the queue artifact's Approve & Send is the only mutation that sends.
     Do-1-first: on the first run in a workspace, draft ONE card and confirm before the full batch. -->

# Prospect — the IB-grade prospecting spine

**Compliance check — step 0 before anything else.**
Read `config/dddb.md`. If the five-point compliance attestation is not complete (all five boxes confirmed, date present), **halt immediately** and tell the buyer: "Compliance attestation in config/dddb.md is not complete. Run `setup` to finish it before prospecting a cold or DDDB source." Do not proceed with a cold source until attestation is confirmed. BYO-list sources with the buyer's own contacts may proceed with a buyer acknowledgment that the same suppression and CAN-SPAM requirements apply.

**Get Attention, pointed forward.** The pile of potential contacts is an asset. This skill works it every run: finds the qualified ones, assigns the right value strategy, drafts on-voice research-backed outreach, and queues it for your approval. Every draft is stamped for split-testing so the improvement loop can learn which strategy wins by sector.

Import `deal-voice` before drafting. Ground every factual claim in `config/products.md` and the active sector pack — never invent a rate, term, or proof point.

## The one rule that matters most

A prospect who doesn't fit your offering is not a lead to send to — it's a waste of a send and a burn of trust. Before anything drafts, every contact passes the qualification gate (`references/qualification-gate.md`). If it fails, it is **excluded silently** — never queued. **We prospect fits, not lists.**

## Triggers

**Fire this skill when …**
- the user says "prospect", "find leads", "build outreach", "work my list", "generate pipeline", "tee up some drafts", "who should I reach out to", "run the prospecting engine".
- a scheduled prospecting task fires.
- the user has an empty outreach queue and wants to fill it.

**Do NOT fire when …**
- config/ is incomplete or `config/dddb.md` attestation is missing — redirect to `setup`.
- it's a reply to an existing thread (that's a different triage/follow-up skill).
- a contact is already in an active deal stage or owned by a live teammate thread.
- a contact is unsubscribed or opted out.

## Architecture: spine + pluggable layers

This skill is the orchestration spine only. It does not hard-code where prospects come from or how value is added. It calls:
- A **SOURCE ADAPTER** (default: `dddb`; fallback: `byo-list`; see `references/sources/`) for the input prospect list.
- A **STRATEGY** (see `references/strategies/`) for the research brief and draft shape.
- The active **SECTOR PACK** (see `references/sectors/`) for domain knowledge and language.

All three are interchangeable. Swap a module file, not the spine. Every draft is stamped `{source_id, strategy_id, sector, voice_version}` for split-testing and logged to the buyer's Supabase schema.

## Strategy assignment (A/B rotation)

Assign strategies by **even round-robin** across the prospect batch: `ib-comp-book`, `proof-led`, `timing-led`, `product-reframe`. For a batch of N prospects, each strategy gets ceil(N/4); distribute the remainder sequentially. Log `strategy_id` on every draft row — that stamp is what makes A/B testing measurable. See `references/ab-testing.md` for the full split-test mechanics.

Sector pack: load the pack matching the prospect's NAICS (see `references/sectors/`). If no sector pack exists, proceed with general `config/products.md` language; note "no sector pack loaded" in the log.

## The loop — do these in order

**0. Compliance gate (see above).** Halt on missing attestation.

**1. Load the source adapter.**
Read `references/sources/dddb.md` (or `references/sources/byo-list.md` if the buyer is using their own list). Pull the normalized prospect list: `{prospect_id, firm, contact_name, role, email, naics, industry, size_band, geo, source, consent_status, provenance_id, collected_at}`. The DDDB pull uses the configured connector (endpoint + credential from `config/dddb.md`; credentials are environment variables, never read from files). Run the five-point compliance gate from `references/sources/dddb.md` before returning any contact — any contact that fails any check is excluded unconditionally.

**2. Run the qualification gate — `references/qualification-gate.md`.**
For each prospect, decide `qualified` / `exclude` / `needs-human-read`. Exclude (don't queue): off-limits, does-not-fit, no-path-to-product, cooldown. Write a one-line skip reason for every exclusion.

**3. Dedup and cooldown.**
Drop any contact already queued with status `pending` or `sent` inside the 30-day cooldown window, or any contact already in an active deal stage. Flag for the rep: any contact already emailed today who should also receive a same-day call (two-touch rule).

**4. Assign strategy + load sector pack.**
For each surviving prospect: assign a `strategy_id` via round-robin (see above). Load the matching strategy file from `references/strategies/<strategy_id>.md` (conditional read: only load strategies assigned to at least one prospect this batch). Load the sector pack from `references/sectors/<sector>.md` if it exists.

**5. Research each survivor — spawn a sub-agent (Task tool).**
Read `references/research-base.md` for shared sub-agent conventions. One researcher per prospect (parallel). Pass: firm (internal only — never in output), NAICS, plain-English industry, size band, state/metro. The strategy file specifies what the researcher returns; the spine passes that brief to the Task. Contact PII stays out of the repo and out of logs.

**6. Draft — call the assigned strategy.**
The strategy file (`references/strategies/<strategy_id>.md`) specifies the angle-selection logic, draft shape, and guardrails. Follow it. Import `deal-voice` before drafting. Lead with the gift, not the ask. One product per touch.

**7. Stamp every draft.**
Before queuing, stamp: `{source_id: "<adapter>", strategy_id: "<assigned>", sector: "<naics-pack or 'general'>", voice_version: "<v>"}`. These four fields are written to every log row (see `references/logging.md`).

**8. Render the approve-and-send queue — `assets/queue.html`.**
One card per prospect: who, fit rationale, chosen angle (editable), research hooks, draft (editable), one customization point for the sender to fill in, and Approve & Send. ~15 cards per run. **Everything is draft-only — the sender's Approve & Send is the only thing that sends.** On the very first run in a workspace, draft **ONE** card and confirm voice and angle before generating the full set (do-1-first).

**9. Log to Supabase — `references/logging.md`.**
- `prospect_events` on each draft (includes strategy_id, source_id, sector stamps).
- `prospect_queue` row per card (status `pending`); exclusions get a `skipped` row.
- `email_drafts` on each send: `draft_initial` vs `final_sent` + `was_edited` + `edit_distance`.

**10. Write the success test — `references/success-test.md`.**
On send, insert one `skill_tests` row. Hypothesis: prospect engages within 7 days. Include `strategy_id` and `sector` in the row so the loop can score by arm.

**11. Handoff.**
When a queued prospect replies, advance it. Hand to the next stage with a one-line note: the strategy that worked, the angle, the lead score. That note is the next stage's starting context. Routing follows `config/routing.md` if configured.

## Reference files (conditional reads — load only what this run needs)

| File | When to read |
|---|---|
| `references/qualification-gate.md` | Always — step 2. |
| `references/logging.md` | Always — step 9. |
| `references/success-test.md` | Always — step 10. |
| `references/research-base.md` | Always — step 5. |
| `references/ab-testing.md` | If this is the first run or rotation logic is uncertain. |
| `references/strategies/<id>.md` | Only for strategies assigned to this batch. |
| `references/sectors/<sector>.md` | Only if a matching pack exists for a prospect in this batch. |
| `references/sources/<adapter>.md` | For the active source adapter. |

## Safeguards

- **Draft-not-send** on everything. The queue artifact's Approve & Send is the only mutation.
- **Do-1-first** on the first run in any workspace.
- **No fabrication** — never invent a rate, term, or proof point. Not in the library → not in the draft.
- **The gate is not optional** — a prospect we can't help is never queued.
- **Compliance gate is not optional** — cold source contacts that fail the five-point gate are excluded unconditionally.
- **PII discipline** — contact names, emails, and exact amounts never enter the repo or logs beyond what the queue artifact requires for the send.

## Output

Prospect-queue artifact (~15 researched cards per run) + Supabase rows. End your turn with a one-line status: "N qualified drafts queued for your approval, M excluded (no-fit / cooldown / compliance), strategy split: ib-comp-book X / proof-led X / timing-led X / product-reframe X, top sector: <x>."
