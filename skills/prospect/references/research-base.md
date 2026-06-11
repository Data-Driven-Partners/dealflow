# Research sub-agent conventions (shared baseline)

All prospecting strategies share a common research layer. This file defines how the researcher sub-agent is spawned, what it returns, and the hard rules that apply regardless of which strategy runs. Each strategy file (`references/strategies/<id>.md`) references this baseline and may narrow or extend the brief for its specific needs.

## The researcher sub-agent (Task tool — one per prospect, parallel)

Spawn a researcher per surviving prospect. Keep them tightly scoped — they return findings; they do not draft or send.

**Standard brief template** (adapt per strategy):
> Research `<company>` (`<city, state>`, `<industry/NAICS>`) for a prospecting outreach email from a deal-maker. Return, with a source link for each: (1) a current company read — size, recent news, hiring/expansion, new locations, contract wins, reviews/ratings, ownership changes; (2) one near-term **industry** trend or pressure affecting businesses like theirs; (3) any **timing signal** that makes a deal, financing, or advisory engagement relevant now (growth, seasonality, a market shift, a regulatory change). Then propose **2–3 specific value hooks** — each one sentence, tied to a finding. No generic hooks. If you find nothing concrete, say so plainly — do not invent.

**Tools available to the researcher:** WebSearch + web_fetch.

**Output destination:** findings are returned to the spine and passed to the assigned strategy for angle selection and drafting. They do not go into logs or the repo directly.

## Hard rules (apply to every researcher, every strategy)

1. **No fabrication.** "Nothing found" beats a made-up detail. A weak-but-true hook beats a strong-but-fake one. If research returns nothing concrete, the strategy must fall back to honest framing — it never invents specifics.

2. **PII discipline.** Contact names, phone numbers, SSNs, and exact named amounts must not enter the repo, logs, or any persisted output. The company name is used internally for research but must not appear in any logged row or artifact text that could be read outside the session. Research findings live only in the draft the buyer approves.

3. **Cite sources.** Every returned finding must carry a source URL or clear source attribution so the buyer can sanity-check before sending. Unsourced claims are treated as unverified.

4. **Scope discipline.** The researcher's job is to find facts that make an angle *true* and *specific*. It does not select the angle, write the draft, or make product recommendations. Those are the strategy's job.

5. **One researcher per prospect.** Spawn them in parallel (Task tool supports concurrency). Do not wait for one to finish before spawning the next.

## Strategy-specific extensions

Each strategy file specifies an **extended brief** that adds to or narrows this baseline. For example:
- `ib-comp-book` replaces standard web research with public comp data lookup (no web search for individual deal specifics).
- `timing-led` requires the researcher to source and cite a real industry/rate signal (not a generic claim).
- `proof-led` directs the researcher to match the prospect to the buyer's proof library first.
- `product-reframe` focuses the researcher on the prospect's current product-fit gap.

Read the strategy file to get the correct brief before spawning.
