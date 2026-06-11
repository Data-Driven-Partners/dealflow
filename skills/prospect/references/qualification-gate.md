# Qualification gate — fit vs can't-help

The single most important step in `prospect`. A contact is only worth outreach if you can **actually help** them. Reaching people you have no product for raises hope, wastes a send, and burns trust. **When in doubt, leave it out** (or mark `needs-human-read`) — never assume qualified.

Evaluate each contact against three questions, in order. The first hard FAIL excludes it.

## 1. Is the contact off-limits? (hard exclude — never queue)

- **Unsubscribed / opted-out / "stop / take me off."** Suppressed forever, no exceptions.
- **Already active** — in an open deal stage or an owned live thread. Hand to the owner; do not re-touch.
- **Inside the cooldown window** (contacted in the last 30 days from this source) → wait.
- **Failed compliance gate** (from the DDDB or BYO-list source adapter) → excluded unconditionally.

## 2. Is the "no" a real no, or is there a genuine fit opportunity? (the judgment call)

Not every cold contact is a good prospect. The question is whether the business has a plausible need that your offering can address.

- **FIT → qualifiable.** The business profile (industry, size, stage, situation) maps plausibly to one or more of your offerings (`config/products.md`). Even if they have not yet expressed interest, the fit is real.
- **NO-FIT → exclude.** The profile does not match any offering in `config/products.md`. No product for them, no reason to reach out. Re-opening a no-fit contact wastes the send.

Use the contact's enrichment data (from the source adapter's `context` field) to make this judgment. If data is thin, mark `needs-human-read` rather than assuming fit.

## 3. Can you actually help them today? (product-fit check — ground in config/products.md)

A plausible fit still has to **clear a real path to a product**. Check against `config/products.md` (your offerings, fit criteria, and any hard eligibility rules). Exclude if there is a hard, unfixable miss on the criteria your products require.

If a contact might fit with a different product than the default (e.g., did not qualify for Product A but could use Product B), it is **qualifiable** — re-angle to the product that fits. Product-mismatch is a crack; whole-profile-ineligible is a real no.

## Output of the gate

For each contact, one of:

- **`qualified`** → continue to research + draft. Tag the likely product fit and any re-angle.
- **`exclude`** → drop. Write a one-line skip reason: `off-limits` | `no-fit` | `cant-help` | `cooldown` | `compliance-fail`. Never queued, never drafted.
- **`needs-human-read`** → data too thin to judge. Surface to the buyer as a quick yes/no; do not auto-draft.

Logging skip reasons is not busywork — it is how the buyer learns, over time, how much of their source is genuinely qualifiable and stops the engine from re-running the same bad outreach.
