# A/B split-test mechanics

The prospect loop is designed to run multiple value strategies head-to-head from day one. This file explains what gets tested, how drafts are stamped, where results land, and how the loop shifts allocation toward winners.

## What we're testing

Each **strategy** (ib-comp-book / proof-led / timing-led / product-reframe) is an independent arm. Sector adds a second axis: the same strategy may perform differently across different industries. An **arm** = `strategy × sector`.

## Phase 1 — even split (start here)

Assign strategies by **even round-robin** within each batch. For a batch of N prospects: strategies cycle `ib-comp-book → proof-led → timing-led → product-reframe → ib-comp-book …`. Each strategy gets floor(N/4) prospects; the remainder goes to the first arms in cycle order.

**Why even split first:** you need a fair baseline before declaring a winner. Sending all prospects to one strategy tells you nothing about the others.

## What gets stamped on every draft

Every draft must carry four fields. Write them to `prospect_events.meta` (JSONB) and mirror them on the `skill_tests` row (see `references/logging.md`):

| Stamp | Example | Purpose |
|---|---|---|
| `source_id` | `"dddb"` | Which source adapter produced the prospect. |
| `strategy_id` | `"ib-comp-book"` | Which value strategy generated the draft. |
| `sector` | `"hvac"` | Active sector pack, or `"general"` if none. |
| `voice_version` | `"v1.0.0"` | Prompt version — lets you attribute outcome changes to prompt edits. |

Missing any stamp = arm data is lost for that row. The evals gate enforces >= 99% stamp coverage.

## Where results land

`skill_tests` — one row per sent draft, with the four stamps in `meta`. The `test_sql` checks for an engagement event (reply / stage move) within 7 days. The improvement loop runs the SQL at `due_at` and writes `result = 'pass' | 'fail'`.

## How the loop scores arms (7-day window)

```sql
SELECT meta->>'strategy_id' AS strategy,
       meta->>'sector'       AS sector,
       round(100.0 * sum((result = 'pass')::int) / nullif(count(*), 0), 1) AS engage_pct,
       count(*) AS n
FROM skill_tests
WHERE skill = 'prospect'
  AND result IN ('pass', 'fail')
  AND due_at >= now() - interval '7 days'
GROUP BY 1, 2
ORDER BY 3 DESC;
```

The improvement loop surfaces this table and flags arms with meaningfully higher or lower `engage_pct`. The buyer reviews and adjusts allocation for the next cycle.

## Phase 2 — weighted allocation (when volume supports it)

Condition: >= 50 completed tests per arm. At that point, shift from even split to **weighted random allocation** proportional to each arm's `engage_pct` (epsilon-greedy or Thompson sampling). The spine's strategy-assignment logic reads current weights from a config row in the buyer's Supabase and allocates accordingly.

Until volume supports phase 2, stay with the even split. **Do not guess at winner allocation early** — small samples are noise.

## What to never do

- Do not cherry-pick which strategy gets "the best leads." The split must be blind to outcome when assigned.
- Do not skip the stamp on any draft. A draft without `strategy_id` is invisible to the loop.
- Do not declare a winner on fewer than 50 tests per arm. Low-volume conclusions are vibes.
