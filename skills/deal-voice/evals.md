# deal-voice — eval set (v1)

Merge gate for this skill. Two parts: **trigger activation** (fires on voice/drafting intent, stays quiet for internal/non-voice tasks) and **binary outcome checks** (manual review of draft output quality).

`ambiguous` rows document the boundary and are excluded from the hard gate.

Activation threshold: >= 80% on combined should/should_not rows.

## Part 1 — trigger activation prompts

| kind | prompt | why |
|---|---|---|
| should | "write in our voice" | Direct voice invocation. |
| should | "does this draft sound like us?" | Voice review intent. |
| should | "check this for voice" | Voice review intent. |
| should | "review this outreach draft" | Draft review requiring voice check. |
| should | "prospect skill importing deal-voice before drafting" | Programmatic import — must fire. |
| should | "rewrite this in our tone" | Voice rewrite intent. |
| should_not | "log this event to Supabase" | Internal/system task — no voice styling. |
| should_not | "write a Supabase migration" | Code/technical task — no voice styling. |
| should_not | "send this unsubscribe auto-reply" | Suppression auto-handle — use minimal template. |
| should_not | "update my config/voice.md" | Config edit — routes to setup, not deal-voice. |
| should_not | "what does my config/voice.md say?" | Read-only config inquiry — answer directly. |
| ambiguous | "write an email to a prospect" | Could trigger deal-voice directly or via prospect — document, don't gate. |
| ambiguous | "draft a follow-up to this reply" | Could be triage/follow-up or a voice check — document. |

Activation math: 6 should + 5 should_not = 11 decisive rows. Threshold: >= 80% (at most 2 wrong).

## Part 2 — binary outcome checks (manual review)

Run as a spot-check sample of sent drafts. A version fails the gate if any check regresses.

**Check A — no invented facts or rates.**
Sample 10 sent drafts. Confirm every rate, term, or product claim traces to `config/products.md` or a cited public source. Any invented fact = hard fail.

**Check B — no beliefs sent as facts.**
Sample 10 sent drafts. Confirm no internal convictions, targets, or model-invented claims are asserted to prospects as proven facts. Any belief presented as fact = hard fail.

**Check C — voice matches config/voice.md.**
Sample 10 sent drafts. Confirm the register, tone, and hard prohibitions from `config/voice.md` are reflected. Significant deviation = fail.

**Check D — easy out present.**
Every sent draft must include a low-pressure easy-out line. Sample 10 drafts; confirm >= 9 of 10 include one. Threshold: >= 90%.

**Check E — voice_version is logged.**
Every `prospect_events` row written when deal-voice is loaded must include a non-null `voice_version` matching the current `metadata.version`. Check:
```sql
select count(*) from prospect_events
where action = 'drafted' and voice_version is null;
-- Should be 0.
```
