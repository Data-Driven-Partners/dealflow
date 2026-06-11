# Strategy: proof-led

**Module:** proof-led
**Hypothesis:** A single, specific, anonymized proof point that mirrors the prospect's profile (industry + size + product) is more persuasive than a general pitch — because it makes the deal feel real and routine for a business like theirs.

## Purpose

Lead with ONE anonymized proof point from the buyer's proof library (loaded via `proof-library` skill; see `config/proof-sources.md`) that mirrors the prospect's profile. Honor the provenance tag strictly. The proof point is the gift; the ask follows.

---

## Research brief (before standard web research)

Before running the standard sub-agent research, **first match the prospect to a proof-library entry**:

1. Read the buyer's current proof library (path configured in `config/proof-sources.md`).
2. Find the closest entry: same or adjacent NAICS, same size band, same product type. Prefer entries from the most recent period.
3. If no entry matches within 1 NAICS level: fall back to `ib-comp-book` strategy. Do not fabricate a proof point; use real comp data instead.
4. If a match exists: use it as the anchor. Then spawn the standard researcher (see `references/research-base.md`) for fresh company/news context to make the angle specific to *this business today*.

---

## Angle selection

The proof point is the opening gift. Frame it as market evidence or as the buyer's own experience, depending on the `source` tag in the proof library:

- **`source: MARKET`** → "Businesses shaped like yours close deals like this regularly — a ~[size] [industry] [product] finalized in about [term/timeline]. That's a normal transaction for a business your size."
- **`source: OURS`** → "We recently helped a [industry] business very close to your size close a [product type] — similar structure, finalized in about [timeline]."

**Never flip these.** A MARKET proof point is market evidence. An OURS proof point is the buyer's own. Mixing them is a compliance and trust violation.

---

## Draft shape

```
[Proof point — the gift. One or two sentences, anonymized, specific to their profile.]

[One sentence tying it to their situation — e.g. "Your profile is a close match."]

[Optional: one fresh company-specific hook from the researcher, if strong.]

[Standard CTA: one clear next step, low commitment.]

[Easy out.]
```

Thread to any prior context. One product per touch. Show the math if a number is used (trace to the proof library or `config/products.md`).

---

## Guardrails

- **Only cite what's in the library.** Never invent a deal.
- **Honor the provenance tag.** MARKET = market evidence; OURS = buyer's own. Never claim you closed a MARKET deal.
- **Anonymization absolute.** No names, no exact amounts tied to a named business, no addresses.
- **If no match exists:** fall back to `ib-comp-book` — never use a weak or mismatched proof point as a substitute.
- **No fabrication in the research hook** — if the researcher finds nothing concrete about the company, omit the company-specific line rather than invent one.
