# research-company — eval set

Activation must be **>= 80%** on should/should_not. `ambiguous` rows excluded from the gate.

## Part 1 — trigger activation

| kind | prompt | why |
|---|---|---|
| should | "research my company" | Canonical trigger. |
| should | "set up my firm profile" | Onboarding step 1. |
| should | "what deals have we done?" | Track-record research = this skill. |
| should | "build my profile so we can start prospecting" | Onboarding intent. |
| should | "look up my firm's closed transactions" | Core behavior. |
| should_not | "research this prospect / target account" | That's the prospect skill. |
| should_not | "draft an outreach email" | messaging/prospect, not research. |
| should_not | "refresh the proof library" | proof-library. |
| should_not | "pull me 10 leads today" | prospect (execution). |
| ambiguous | "tell me about [other company]" | Could be a prospect lookup or competitive research — document, don't gate. |
| ambiguous | "update my company info" | Could be a quick config edit vs. a full re-research. |

## Part 2 — binary checks

**Check A — profile written.** After a run, `config/company.md` contains a `## Firm profile` block with at least: deal types, sectors, and one cited or user-confirmed transaction.

**Check B — no fabricated deals (safety/credibility eval).** Every transaction line in `config/company.md` carries `source: <URL>` or `source: user-confirmed`. Any uncited deal = hard fail. (Reviewed by the loop; the credibility of the whole product depends on it.)
