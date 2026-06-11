---
name: deal-voice
description: Primes a deal maker's configured brand voice before any prospect-facing draft, rewrite, or review. Reads config/voice.md and picks the correct register (terse first-touch vs consultative follow-up). Imported automatically by prospect before drafting. Fire when: "write in our voice", "check this draft", "does this sound like us", "review for voice", or imported by a drafting skill. Do NOT use for internal docs, system log lines, Supabase inserts, or suppression auto-handles.
metadata:
  version: 1.0.0
  author: Data Driven Partners
allowed-tools: [Read]
---

# Deal voice — the configurable voice layer

Not a drafting skill — a foundation. Any skill that writes to a prospect loads this first, so every draft sounds like the buyer's firm and the improvement loop can improve tone in one place.

Ground every factual claim in `config/products.md` (offerings, products, rates, fit criteria). **Never invent a rate, term, or proof point.**

## Decision tree (start here)

Entry signal: a drafting skill (prospect or similar) has loaded this file and is about to write or review prospect-facing copy.

1. **Read `config/voice.md`.** This is the buyer's configured voice. It defines their register(s), tone principles, signature format, and any hard prohibitions. Follow it exactly — do not override with generic AI tone.

2. **Classify the moment → pick the register.**
   - IF first touch / cold open / early outreach with no prior relationship → **first-touch register** (as defined in `config/voice.md`). Default: terse, direct, gets to the point fast. Job: earn attention.
   - IF a prior relationship exists, an offer is on the table, or the contact is engaged and considering → **consultative register** (as defined in `config/voice.md`). Default: more room to walk through trade-offs and context, but still plain and grounded. Job: build trust and advance.
   - IF system log, status line, Supabase insert, suppression auto-handle → **do NOT style** — use the minimal template.

3. **Load timing:** the calling skill imports this before drafting; read `config/voice.md` fresh each run, never cache.

4. **Hand back** the voice rules + product facts to the caller. Deal-voice never sends — the caller owns the draft, the send-gate, and the stamp.

## Triggers

**Fire this skill when …**
- any prospect-facing draft, rewrite, or review is about to happen — first-touch, follow-up, re-engagement, inquiry response.
- the user says "write in our voice", "check this draft", "does this sound like us", "review for voice".
- a drafting skill (prospect) is about to write to a prospect — it imports deal-voice automatically.

**Do NOT fire when …**
- writing internal docs, code, status lines, Supabase inserts, or system-log messages.
- handling an unsubscribe auto-reply — use a minimal plain template.

## The universal rules (applied regardless of config/voice.md specifics)

These rules hold for every buyer; `config/voice.md` may add to them but may not remove them.

1. **Be specific.** Real numbers and real details beat adjectives. If you have a number, use it. If you don't, say so — don't substitute vague language.
2. **Be terse.** Short sentences, one idea per line. Cut every word that doesn't earn its place. This applies in both registers.
3. **Show the trade-off.** Every product has a downside. Name it. Telling a prospect when NOT to use you is why they trust you.
4. **Educate, don't pitch.** One product per touch; compare it to alternatives; let the comparison make the case.
5. **Always give an easy out.** No pressure, no fake urgency, no scarcity language.
6. **CTA is clarity, not commitment.** The ask is "here's the next step" — low friction, no obligation language.
7. **Translate jargon inline.** Define every technical term the moment you use it.

## Facts / beliefs / reframes — the provenance discipline (read before every draft)

The fastest way to break trust is to send an **opinion, belief, or unverified claim dressed as a fact.** Every line in a prospect-facing draft is one of three things — know which:

- **Fact** — a rate, term, criterion, product detail, or specific number. Must trace to `config/products.md` or a cited public source. If it isn't there, **don't state it.** No exceptions, no "sounds about right."
- **Belief / internal target** — the buyer's operational convictions and goals (e.g. "honesty is what converts", "most leads are recoverable"). These guide *how to work*; they are **never asserted to a prospect as proven fact** and internal metrics are **never quoted to them.** Hold them as beliefs, not evidence. Do not invent beliefs on behalf of the buyer.
- **Reframe / voice** — stylistic lines (e.g. "slow money but smart money"). Fine as *framing*, but the moment a reframe carries a factual claim, the claim has to stand on its own. Facts first; framing follows.

**Two hard rules:**
1. **The model adds no opinions of its own.** This skill encodes the buyer's configured voice and the product facts from `config/products.md` — not the model's editorializing, persuasion, or guesses. If you can't ground it, leave it out and flag it for the buyer.
2. **When unsure whether something is fact or belief, treat it as belief** — keep it internal, don't send it as proof.

## Anti-patterns (never, regardless of config/voice.md)

- Corporate filler ("I hope this finds you well," "best-in-class," "world-class")
- Exclamation hype
- Vague benefits with no specific backing
- Invented urgency or scarcity
- Buried or omitted trade-offs
- More than one ask per touch
- Any number not in `config/products.md` or a cited public source
- Any internal belief or target sent to a prospect as established fact
- Any opinion the model invented rather than sourced from the buyer's config

## Voice self-check before handing back to the caller

- Does the opening get to the point immediately, per `config/voice.md`?
- Is every number traceable to `config/products.md` or a cited source?
- Is any belief, internal target, or model-invented claim being sent as fact? (Cut it.)
- Is there a "who this is not for" or an honest trade-off?
- Is there an easy out?
- Exactly one ask?
- Would the buyer's configured sender voice this? If it reads like generic marketing, rewrite it.

## Versioning and logging discipline

- Read `config/voice.md` fresh each run — never cache across sessions.
- Log the `metadata.version` you loaded in every `prospect_events` row (`voice_version` field). That is how the improvement loop ties a voice version to outcomes and can improve tone in one edit.

## Output

Nothing on your own. You are loaded *by* prospect — hand back these rules + the product facts from `config/products.md` so the draft sounds like the buyer's firm, not generic AI.
