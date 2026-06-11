# Logging — write to Supabase (the improvement loop's fuel)

Use the buyer's Supabase connector (connection string in environment — see `config/dddb.md` for the setup pointer). Every prospect draft and send **must carry the four split-test stamps**: `source_id`, `strategy_id`, `sector`, `voice_version`. Missing any of them = dark data. Write them to the `meta` JSON column.

The table names below are the recommended schema. Buyers may rename to fit their existing Supabase setup; update these names in the logging calls accordingly.

## On every classify + draft

```sql
INSERT INTO prospect_events (skill, "user", target_prospect_id, action, draft_id, voice_version, workspace, meta)
VALUES (
  'prospect', :user, :prospect_id, 'drafted', :draft_id, :voice_version, :workspace,
  jsonb_build_object(
    'source_id',     :source_id,      -- e.g. 'dddb' or 'byo-list'
    'strategy_id',   :strategy_id,    -- e.g. 'ib-comp-book'
    'sector',        :sector,          -- e.g. 'hvac' or 'general'
    'voice_version', :voice_version
  )
);
```

## On every exclusion (skipped contacts)

```sql
INSERT INTO prospect_queue (skill, target_prospect_id, status, skip_reason, source)
VALUES ('prospect', :prospect_id, 'skipped', :skip_reason, :source_id);
-- skip_reason: 'off-limits' | 'no-fit' | 'cant-help' | 'cooldown' | 'compliance-fail'
```

## On every send (THE GOLD STANDARD CAPTURE — do not skip)

```sql
INSERT INTO email_drafts (target_prospect_id, thread_id, draft_initial, final_sent, was_edited, edit_distance, skill, "user")
VALUES (:prospect_id, :thread_id, :draft_initial, :final_sent, :was_edited, :edit_distance, 'prospect', :user);
-- draft_initial = what the engine wrote; final_sent = what the buyer actually sent. The diff trains the skill.
```

Also mark the queue row sent:

```sql
UPDATE prospect_queue SET status='sent', sent_at=now() WHERE id=:queue_id;
```

## A/B stamp fields — reference

| Field | Example values | Note |
|---|---|---|
| `source_id` | `'dddb'`, `'byo-list'` | Where the prospect came from. |
| `strategy_id` | `'ib-comp-book'`, `'proof-led'`, `'timing-led'`, `'product-reframe'` | Which value strategy was used. |
| `sector` | NAICS-derived label (e.g. `'hvac'`, `'restaurant'`, `'general'`) | Active sector pack, or `'general'` if none loaded. |
| `voice_version` | Semantic version string, e.g. `'v1.0.0'` | Tracks voice/prompt changes across time. |

If the buyer's Supabase does not yet have a `meta jsonb` column, write all four fields into whichever columns are available and note the gap for the next migration. The improvement loop reads `meta->>'strategy_id'` for arm scoring (see `references/ab-testing.md`).

## Minimal schema (if starting from scratch)

```sql
-- Run once in the buyer's Supabase project.
create table if not exists prospect_events (
  id bigserial primary key,
  skill text,
  "user" text,
  target_prospect_id text,
  action text,
  draft_id text,
  voice_version text,
  workspace text,
  meta jsonb,
  created_at timestamptz default now()
);

create table if not exists prospect_queue (
  id bigserial primary key,
  skill text,
  target_prospect_id text,
  status text,
  skip_reason text,
  source text,
  sent_at timestamptz,
  queued_at timestamptz default now()
);

create table if not exists email_drafts (
  id bigserial primary key,
  target_prospect_id text,
  thread_id text,
  draft_initial text,
  final_sent text,
  was_edited boolean,
  edit_distance int,
  skill text,
  "user" text,
  created_at timestamptz default now()
);

create table if not exists skill_tests (
  id bigserial primary key,
  skill text,
  stage text,
  target_prospect_id text,
  "user" text,
  hypothesis text,
  test_sql text,
  due_at timestamptz,
  result text,
  meta jsonb,
  created_at timestamptz default now()
);
```
