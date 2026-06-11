# Strategy: ib-comp-book

**Module:** ib-comp-book
**Hypothesis:** A prospect who receives a tight, data-backed "deals of this shape close regularly" mini comp book — as a gift before any ask — will engage at a higher rate than one who receives a generic outreach.

## Purpose

Research comparable transactions from public data (or the buyer's own proof library), assemble a tight "deals like yours happen all the time" mini comp book, and lead the outreach with that comp book as the value gift. The comp book is specific, sourced, and anonymized — it makes the deal feel real and routine for a business shaped like theirs.

---

## 1. Sub-agent research brief

Pass the target descriptor to the researcher (see `references/research-base.md` for baseline rules). Extended brief for this strategy:

> For `<firm>` (internal only — never in output): NAICS `<naics>`, `<plain-English industry>`, estimated size band `<band>`, `<state/metro>`. Pull comparable transaction patterns from the buyer's proof library (`config/proof-sources.md`) AND any applicable public data sources listed there — no web search for individual deal specifics. Apply the 4-axis comp-selection logic below. Return a single ≤150-word text block (the COMPARABLE TRANSACTION PATTERNS block) ready to paste into the outreach draft.

**Source constraint:** use only sources listed in `config/proof-sources.md`. If the sector's flagship sector pack (`references/sectors/<sector>.md`) references a public data source, use that. No fabricated comps.

**Output destination:** the comp-book text block, embedded as the opening gift in the draft.

**Tone:** value gift to a decision-maker — plain language, no jargon, no fabricated numbers.

---

## 2. Comp-selection logic (4 axes, in order)

**Axis 1 — Industry/NAICS (soft):** match same 3-digit NAICS prefix. 2-digit acceptable for thin verticals; flag as "approximate" in the output.

**Axis 2 — Deal/transaction size band (hard):** four bands (or buyer-configured equivalent):
- $50k–$150k
- $150k–$500k
- $500k–$2M
- $2M–$5M

The comp MUST be in the same band as the prospect's estimated size. Do not cite a $2M comp to a $100k prospect.

**Axis 3 — Product/program (hard):** match the product type. Never cite a comp from a different product category than what is relevant for this prospect.

**Axis 4 — Geography (soft):** same state preferred → expand to region → national (note "nationally" if national). Prefer recency (most recent available data).

**Target:** 3–5 patterns. If fewer than 3 are available, use what exists with an honest note ("narrower market — 2 comparable patterns found"). Never synthesize or invent a comp to hit the count.

---

## 3. Output / deliverable shape

A single **≤150-word text block** titled:

```
COMPARABLE TRANSACTION PATTERNS — [industry], [region]
Data source: [source name and vintage, e.g. "SBA 7(a) FOIA dataset, FY2023–2024"]
```

Followed by 3–5 anonymized entries, each on one line:
```
[Industry descriptor] | [size band] | [Product/program] | [Term or structure] | ~[rate or return range from source] | Approx [year] | [Any notable metric e.g. jobs, revenue multiple]
```

Closed with:
```
What this means for you: [1–2 plain sentences — the deal type is real and routine for a business shaped like theirs.]
```

This block is the **opening gift before any ask** in the draft.

---

## 4. Draft shape

```
[Comp book block — the gift]

[One sentence connecting the comp to their specific situation.]

[Standard CTA: one clear next step, low commitment. E.g. "Worth a 5-minute conversation? Reply and I'll show you what a deal shaped like yours actually looks like — no application, no obligation, just clarity."]

[Easy out: "If the timing's off, no worries — happy to circle back when it makes sense."]
```

Thread to any prior context so it reads as continuation, not a cold drop.

---

## 5. Hard guardrails

- **No fabricated comps.** Every entry maps to a real source (public data, the buyer's proof library). Thin data → fewer comps + honest note. Never synthesize a comp that doesn't exist.
- **MARKET data is NEVER "we closed this."** Public data comps are market evidence ("deals of this shape close regularly"). Only proof-library entries tagged `source: OURS` (the buyer's own closed deals) may use "we closed."
- **Anonymization is absolute.** No names, no name-linked exact amounts, no addresses, no identifying details.
- **Cite the source line always.** Every comp block must carry the data-source attribution.
- **Rates only from config/products.md or cited public data.** Omit rate if not in those sources; note "rate subject to market pricing." Never invent a rate.
- **No "approved" / "pre-qualified" language.** The comp book shows the deal TYPE is real and routine — not that the prospect will qualify.
