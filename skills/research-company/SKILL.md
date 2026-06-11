---
name: research-company
description: Researches the Dealflow user's OWN firm to seed their prospecting profile — scrapes their website and searches the web for their closed transactions, deal track record, focus areas, and positioning, then writes a confirmed company profile that drives targeting, sample assets, and messaging. Fire on first-run onboarding or when the user says "research my company", "set up my profile", "what deals have we done", "build my firm profile". Do NOT use to research a prospect or target account (that's the prospect skill), to draft outreach, or to invent deals not found in real sources.
metadata:
  version: 1.0.0
  author: Data Driven Partners
  stage: "Dealflow · onboarding (step 1 of 4)"
allowed-tools: [WebSearch, Read, Write]
---

# Research company — the front door

Step 1 of Dealflow onboarding. Before we target anyone, we learn **who the user is** — their firm, the deals they actually win, their edge — so every prospect and every message is grounded in their real track record, not generic claims. Output is a confirmed profile in `config/company.md` that the strategy, sample-assets, and messaging skills all read.

Ground everything in **real, cited sources**. **Never invent a transaction, client, or credential.** A "couldn't confirm" beats a fabricated deal — the whole product runs on credibility.

## Decision tree (start here)

1. **Get the firm's domain** — ask the user for their website if not known.
2. **Read the website** — home, about, team, services, and any "transactions / deals / case studies / news" pages.
3. **Search the web** — the firm's closed-deal footprint and positioning (see step 2 of the loop).
4. **Synthesize → draft the profile**, show it to the user, **get confirmation/edits**.
5. **Write** the confirmed profile to `config/company.md`; hand off to `outreach-strategy`.

## Triggers
**Fire when** onboarding starts, or the user says "research my company", "set up my profile", "build my firm profile", "what deals have we done".
**Do NOT fire** when researching a *prospect/target* (use `prospect`), drafting outreach (`messaging`/`prospect`), or refreshing the proof library (`proof-library`).

## The loop

**1. Read their site.** Fetch the homepage + about/team/services + any transactions/case-studies/news pages. Pull: what they do, who they serve, deal/transaction types, sectors, typical sizes, team/seniority, and the positioning language they use about themselves.

**2. Search the web for their track record.** Run targeted queries and keep only sourced hits:
- `"<firm>" (advised OR represented OR closed OR acquisition OR financing OR merger OR sale)`
- `"<firm>" transaction OR deal OR "served as"`
- `"<firm>" press release OR news <recent years>`
- principals' names + "advised/closed" where the firm is small.
Capture each closed transaction with its **source URL**, date, parties (only as publicly reported), type, and size if stated. If little is public, say so plainly — do not pad.

**3. Synthesize the profile** (`references/profile-schema.md` for the shape): firm summary; what they do; **deal types / transaction types**; **sectors/industries**; **typical deal sizes / stages**; geography; notable **closed transactions** (cited); positioning/edge; and an initial read on who they likely want to reach (seed for `outreach-strategy`).

**4. Confirm with the user.** Show the draft profile. Ask them to correct/add — especially deals not public and their real focus. Nothing is authoritative until they sign off (same spirit as draft-not-send).

**5. Write `config/company.md`** with the confirmed profile, then hand to `outreach-strategy` with a one-line summary of their focus.

## Guardrails
- **No fabricated deals/clients/credentials** — every transaction cites a real source or comes from the user's own confirmation (tag which).
- **Public sources only** for track record; the user supplies anything private and owns it.
- **Accuracy over completeness** — a short true profile beats an impressive false one.

## Output
A confirmed firm profile in `config/company.md` (summary, deal types, sectors, sizes/stages, geography, cited closed transactions, positioning) + a one-line handoff to `outreach-strategy`.
