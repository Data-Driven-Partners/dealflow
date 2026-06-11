# Success test — how prospect checks its own work

When the buyer sends a draft, the skill writes ONE row to `skill_tests` describing how we will know, later, whether this outreach worked. The improvement loop runs the SQL at `due_at` and records pass/fail.

## What to write (on each send)

```sql
INSERT INTO skill_tests (skill, stage, target_prospect_id, "user", hypothesis, test_sql, due_at, meta)
VALUES (
  'prospect', 'Outreach', :prospect_id, :user,
  'Prospect engages (reply / meeting booked / stage move) within 7 days',
  -- deterministic check, run at due_at:
  'SELECT (count(*) > 0) FROM email_activity_events e
     WHERE e.prospect_id = ''' || :prospect_id || '''
       AND e.direction = ''received''
       AND e.ts > timestamp ''' || :sent_at || '''',
  (timestamp :sent_at + interval '7 days'),
  -- A/B stamps — required for arm scoring:
  jsonb_build_object(
    'strategy_id',   :strategy_id,
    'source_id',     :source_id,
    'sector',        :sector,
    'voice_version', :voice_version
  )
);
```

Adapt the `test_sql` to the buyer's actual engagement tracking table and column names.

## Rules

- The hypothesis is plain English; the `test_sql` returns a single boolean.
- Pick a `due_at` that is fair for the stage (cold outreach = 7 days; adjust if the buyer's sales cycle is longer).
- Keep tests about *observable outcomes* (a reply, a stage change, a booked call) — not vibes.
- **Always include `strategy_id` and `sector` in `meta`** — the improvement loop scores engagement **by arm** (`strategy × sector`), not overall.

## How the loop reads arm results (7-day window)

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

This is the engine of the bandit: arms with higher `engage_pct` get more allocation in the next cycle. Start with even round-robin (see `references/ab-testing.md`); graduate to weighted allocation when volume supports it (>= 50 results per arm).

## Honest caveat

A single per-send test is noisy — one reply proves little. Its value is in **aggregate**: the loop reads many of these and looks at pass-rate by arm, not any one row. Treat the per-row result as a data point, never a verdict.
