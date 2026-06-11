# Source adapter: dddb

**Adapter ID:** `dddb`
**Status:** primary source. Requires completed compliance attestation in `config/dddb.md` before activation. Do not activate until attestation is confirmed and credentials are configured.

---

## Purpose

Pull cold prospects from DDDB — a contact database held in the operator's **Supabase** instance — and feed them into the prospect spine as a fresh outreach list. DDDB contacts are the product wedge: buyers get vetted, enriched contacts AND the research-and-drafting engine that turns them into IB-grade outreach. The adapter reads from a Supabase table via the Supabase connector (no separate API).

**Critical distinction from a warm source:** these are COLD prospects. They have no prior relationship with the buyer. That changes the compliance requirements entirely.

---

## Normalized prospect record (output schema — same across all adapters)

```json
{
  "prospect_id":    "<DDDB contact ID>",
  "firm":           "<company name — INTERNAL ONLY, never in logs/output>",
  "contact_name":   "<primary contact name — internal only>",
  "role":           "<title/role>",
  "email":          "<email address>",
  "naics":          "<NAICS code>",
  "industry":       "<plain-English industry label>",
  "size_band":      "<one of: $50k–$150k | $150k–$500k | $500k–$2M | $2M–$5M>",
  "geo":            "<state, metro>",
  "source":         "dddb",
  "provenance_id":  "<DDDB provenance record ID>",
  "consent_status": "<opt-in | none>",
  "collected_at":   "<ISO timestamp of data collection>"
}
```

All fields are required. A contact missing `provenance_id` or `consent_status` fails the compliance gate unconditionally.

---

## Five-point hard compliance gate — MANDATORY before any contact is returned to the spine

This gate runs on every contact before it is returned. It cannot be disabled. Any contact that fails any check is excluded and must not enter the prospect queue.

**Check 1 — Provenance present.**
The contact must have a documented DDDB provenance record (`provenance_id` must be non-null and must resolve to a real provenance entry that describes where, when, and how the data was collected). No provenance record → excluded.

**Check 2 — CAN-SPAM compliance.**
Every email sent via this adapter must: (a) identify the sender clearly by name and business, (b) include a physical mailing address, (c) include a working unsubscribe/opt-out mechanism, (d) honor opt-out requests within 10 business days. The sending infrastructure must support these requirements before this adapter is activated.

**Check 3 — Suppression check.**
Cross-reference the full contact list against the buyer's suppression list (loaded from config or the buyer's CRM/Supabase) before returning any prospect. Any contact on the suppression list is excluded unconditionally. The suppression list must include all prior opt-outs, do-not-contact requests, and any other buyer-maintained exclusions.

**Check 4 — Consent status.**
`consent_status` must be `opt-in` or explicitly cleared by DDDB's consent records. `consent_status = 'none'` is not sufficient for cold commercial outreach in jurisdictions with stricter standards. Contacts with `consent_status = 'none'` are **flagged for the buyer's legal review, not auto-queued**. The buyer must affirmatively decide to include them after their own legal review.

**Check 5 — Jurisdiction.**
Some jurisdictions (California/CPRA, Colorado, Virginia, Connecticut, EU/GDPR, Canada/CASL, and others) have additional commercial email, data-broker, and consent requirements. Contacts in flagged jurisdictions require additional review before contact. Flag by state/country; do not auto-include without buyer review.

**Any contact that fails any check is excluded and must not enter the prospect queue.**

---

## Pull logic (Supabase — operator configures)

The adapter queries the **Supabase contacts table** named in `config/dddb.md`, via the Supabase connector, using the service key referenced there by env-var name. Never read or commit the key itself.

Query the contacts table with these filters (passed from the spine), mapping table columns per the column-map in `config/dddb.md`:
- `naics_filter`: target NAICS codes or prefixes.
- `size_band_filter`: one or more of the four size bands.
- `geo_filter`: state(s) or metro(s).
- `recency`: `collected_at` cutoff.
- `limit`: max rows per run.

Then: left-anti-join against the **suppression table**, apply the five-point compliance gate to every remaining row, and return only rows that pass all five checks as the normalized record schema above. Rows missing `provenance_id`, `consent_status`, or `collected_at` fail unconditionally.

---

## Credential and table configuration

Set in the operator's Supabase connector + `config/dddb.md`:
- Supabase project ref / URL and the **env-var name** holding the service key (never the key itself).
- The contacts table and suppression table (schema.table).
- The column map (table columns → normalized fields) and default filters.

**Never commit a Supabase key or secret to any file.** If one is accidentally committed, rotate it immediately.

---

## Legal notice

**The buyer must obtain their own legal review before activating this adapter.** The plugin enforces the mechanical compliance checks described above; it does not guarantee legal sufficiency in any jurisdiction. CAN-SPAM, CPRA, GDPR, CASL, and other regulations impose obligations on the sender — not just the tool. Consult qualified legal counsel before contacting cold prospects in any jurisdiction.
