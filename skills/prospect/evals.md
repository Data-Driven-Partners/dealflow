# prospect — eval set (v1)

Merge gate for this skill. Two parts: **trigger activation** (fires on prospecting intent, stays quiet otherwise) and **binary outcome checks** (deterministic SQL against the buyer's Supabase schema). The ship gate and the weekly loop read this file: activation must be >= 80% on `should`/`should_not`, and no binary check may regress vs the prior version.

`ambiguous` rows document the boundary and are excluded from the hard gate.

## Part 1 — trigger activation prompts

| kind | prompt | why |
|---|---|---|
| should | "prospect for me" | Canonical trigger. |
| should | "find leads in the HVAC sector" | Sector-specific prospecting intent. |
| should | "work my list" | BYO-list prospecting intent. |
| should | "generate pipeline" | Pipeline-building = prospecting. |
| should | "tee up some outreach" | Prospecting-queue intent. |
| should | "I have no outreach queued, fill the pipeline" | Empty-queue → prospect. |
| should | "run the prospecting engine" | Direct engine invocation. |
| should | "pull DDDB contacts and draft outreach" | DDDB source + draft intent. |
| should | "build outreach using the comp-book strategy" | Strategy-aware trigger — must fire. |
| should | "prospect HVAC leads with the timing angle" | Strategy-aware trigger — must fire. |
| should_not | "set up my config" | Setup intent — routes to setup. |
| should_not | "triage my replies" | Inbound triage — routes to a triage skill. |
| should_not | "send the quote to this contact" | Quote/send stage — different skill. |
| should_not | "this contact unsubscribed, what do I do?" | Suppression action — not prospecting. |
| should_not | "update the proof library" | Proof-library maintenance — routes to proof-library. |
| ambiguous | "follow up with contacts who didn't reply" | Could be prospect (cold re-touch) or triage (warm thread) — document, don't gate. |
| ambiguous | "who should I reach out to today?" | Could trigger prospect or be answered by reviewing an existing queue — document, don't gate. |
| ambiguous | "build me a prospecting list" | List-building vs working the list — adjacent but not identical. |

Activation math: 10 should + 5 should_not = 15 decisive rows. Threshold: >= 80% (at most 3 wrong).

## Part 2 — binary outcome checks

Run against the buyer's Supabase after a representative window. A version fails the gate if any check regresses vs the prior version. Table names below follow the schema in `references/logging.md`.

**Check A — engagement rate within 7 days (the core prospecting success test).**
Threshold (example baseline): >= 10% of prospect success tests pass.
```sql
select round(100.0 * sum((result = 'pass')::int) / nullif(count(*), 0), 1) as engage_7d_pct
from skill_tests
where skill = 'prospect'
  and result in ('pass', 'fail');
```

**Check B — the qualification gate is actually filtering.**
Threshold: exclusion share >= 10% of contacts evaluated in the window (proves the gate is doing work).
```sql
select round(100.0 * sum((status = 'skipped')::int) / nullif(count(*), 0), 1) as excluded_pct
from prospect_queue
where source in ('dddb', 'byo-list');
```

**Check C — no fabricated proof points.**
Every proof point cited in a sent draft must trace to the buyer's proof library or a cited public source. Manual/spot review: sample sent drafts referencing "a similar deal" and confirm the deal exists in the proof library or carries a source URL. Any invented proof = hard fail.

**Check D — strategy_id is being logged on every draft.**
Every sent draft must carry a non-null `strategy_id`.
```sql
select round(100.0 * sum((meta->>'strategy_id' is not null)::int) / nullif(count(*), 0), 1) as strategy_stamped_pct
from prospect_events
where action = 'drafted';
-- Threshold: >= 99%
```

**Check E — strategy distribution is roughly even.**
Each strategy should receive between 15% and 35% of drafts in a window of >= 40 drafts.
```sql
select meta->>'strategy_id' as strategy, count(*) as n,
       round(100.0 * count(*) / sum(count(*)) over (), 1) as pct
from prospect_events
where action = 'drafted'
group by 1 order by 2 desc;
-- Alert if any strategy < 15% or > 35% in a window of >= 40 drafts.
```

**Check F — compliance gate ran (no non-attested DDDB contact was queued).**
Manual check: confirm `config/dddb.md` attestation date predates the earliest DDDB-sourced `prospect_queue` row. Any DDDB row with a `queued_at` before the attestation date = hard fail.

## Notes

- Check B is the safety eval: proves the qualification gate is doing its job, not rubber-stamping every contact.
- Checks D and E are split-test integrity evals: without them, the A/B data is untrustworthy.
- Check F is the compliance integrity eval: non-negotiable for a sold contact source.
