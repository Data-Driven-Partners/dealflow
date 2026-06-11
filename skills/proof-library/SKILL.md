---
name: proof-library
description: Builds and maintains the buyer's proof library — anonymized deal or transaction patterns that prospect and deal-voice cite to build trust. Pulls from the buyer's own closed deals (tagged OURS) and optional public sector data sources (tagged MARKET). Fire when: "build the proof library", "refresh proof points", "update closed deals", "add proof points", or the scheduled weekly task runs. Do NOT use to publish contact names, invent deals, or claim you closed a deal sourced from public data.
metadata:
  version: 1.0.0
  author: Data Driven Partners
allowed-tools: [WebSearch, Read, Write, Supabase]
---

# Proof library — the trust repository

Deal makers convert on proof, not adjectives. The strongest outreach line is "a business shaped exactly like yours recently did this." This skill keeps a living repository of those reference patterns — **anonymized, structured, and provenance-tagged** — so every skill can cite a real, true pattern instead of a vague claim. It runs on a schedule and grows over time; the bigger the library, the sharper every draft.

Output lives at the path configured in `config/proof-sources.md` (default: `knowledge/proof-library.md`) — version-controlled and safe to share.

## The honesty rule (read first — this is a compliance line)

Two kinds of proof, **never blurred**:

- **OURS — `source: OURS`.** The buyer's own closed or completed transactions, anonymized. Only these may be phrased *"we closed"*, *"we completed"*, *"we helped a [industry] business."*
- **MARKET — `source: MARKET`.** Public data across all participants (e.g. a public FOIA dataset, an industry report). Phrase as *"deals of this shape close regularly"* / *"a ~[size] [industry] transaction of this type was completed in [period]."* **Never** phrase as "we closed this" — it's market evidence the deal type is real and routine, not the buyer's own work.

Every entry carries its `source` tag, and consuming skills (prospect, deal-voice) must honor it. **Never invent a deal, a rate, or a timeline. Never publish a contact or counterparty name, exact amount tied to a name, or any PII.** Round to bands.

## Decision tree (start here)

**Entry signal:** the weekly proof-library task runs, or the buyer asks to refresh proof points.

1. **Pull MARKET data** from public sources listed in `config/proof-sources.md`. Check each source for the most recent available data. If a sector pack (e.g. `references/sectors/sba-smallbiz-lending.md`) lists public data sources for that sector, use those.
2. **Filter to the buyer's relevant deal shapes** — the industries, size bands, product types, and geographies described in `config/products.md`.
3. **Aggregate into anonymized patterns** — group by industry × size-band × product-type × region; never per-named-business. Derive typical structure (term, rough rate range from `config/products.md`, typical timeline). Do not store identifying information.
4. **(Optional) Add buyer's own closed/completed deals** — pulled from their CRM or a provided list, tagged `source: OURS`. Anonymize: round amounts to bands, omit names, omit addresses. The buyer confirms which deals to include.
5. **Write or merge into the proof library file** (path from `config/proof-sources.md`) — upsert entries, keep the schema, bump the "last updated" line. Do not duplicate; refresh existing patterns with newer data.
6. **Log** one `prospect_events` row (skill `proof-library`, action `updated`, count of entries) — store no PII.

## Entry schema (one entry per pattern)

```yaml
- naics: "238220"
  industry: "HVAC & Mechanical Contractors"
  size_band: "$150k–$500k"
  product_type: "[product type from config/products.md]"
  region: "Southeast"
  structure: "[typical term or structure, e.g. '10-yr term, fixed rate']"
  rate_range: "[from config/products.md, e.g. '~10–12%']"
  period: "FY2023–2024"
  source: "MARKET"                       # MARKET or OURS — never omit
  data_source_url: "[url if MARKET]"     # required for MARKET; omit for OURS
  note: "[optional: any qualification nuance or anonymization note]"
  added: "2026-06-09"
```

OURS entries: omit `data_source_url`; include an internal reference ID the buyer can use to verify (not stored in any public output).

## Triggers

**Fire this skill when …**
- the **weekly proof-library** scheduled task runs.
- the buyer says "build/refresh the proof library", "update closed deals", "what deals can I reference", "add proof points".

**Do NOT fire when …**
- drafting a specific prospect email (that's prospect — it *reads* the library; it doesn't rebuild it).
- asked to expose real contact identities or exact named amounts — refuse; the library is anonymized by design.

## Sources

Sources are configured in `config/proof-sources.md`. Examples:

- **Public sector FOIA data** — for sectors like SBA small business lending, the SBA FOIA dataset (data.sba.gov/en/dataset/7-a-504-foia) provides public deal patterns. Always check the dataset page for the current file URL (the `asof` date changes quarterly).
- **Buyer's own CRM export** — closed/won deals from the buyer's CRM, anonymized as above.
- **Industry reports** — publicly available aggregate deal data (cite the source and vintage).

For each MARKET source: note the URL, the data vintage, and what it covers. Ground all rate/term framing in `config/products.md` — public datasets give shapes and amounts, the buyer's products config gives the honest rate language.

## Output

Updated proof library file + a one-line status: "Proof library updated: N market patterns (MARKET), M our-deals (OURS). Top new shapes: [list]."
