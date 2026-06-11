# Source adapter: byo-list

**Adapter ID:** `byo-list`
**Status:** secondary source. Buyer uploads a CSV or connects their CRM. Same normalized record schema as `dddb`; same suppression and CAN-SPAM requirements apply. No external DDDB compliance attestation required, but the buyer must confirm that their own list meets the same five compliance points.

---

## Purpose

Accept a buyer-supplied contact list (CSV upload or CRM export) as the input prospect source. The buyer owns this data — they are responsible for how it was collected and whether it meets applicable legal standards. The engine applies the same qualification gate, research, strategy assignment, and drafting logic as it does for DDDB contacts.

---

## Normalized prospect record (output schema — same across all adapters)

```json
{
  "prospect_id":    "<buyer-assigned ID or row number>",
  "firm":           "<company name — INTERNAL ONLY>",
  "contact_name":   "<primary contact name — internal only>",
  "role":           "<title/role>",
  "email":          "<email address>",
  "naics":          "<NAICS code — required for sector-pack matching; buyer must supply or the engine will attempt lookup>",
  "industry":       "<plain-English industry label>",
  "size_band":      "<one of: $50k–$150k | $150k–$500k | $500k–$2M | $2M–$5M>",
  "geo":            "<state, metro>",
  "source":         "byo-list",
  "provenance_id":  "<buyer reference — how this contact was collected>",
  "consent_status": "<opt-in | none | buyer-attested>",
  "collected_at":   "<ISO timestamp or approximate date>"
}
```

---

## CSV column mapping

If the buyer supplies a CSV, map columns to the normalized schema above. Required columns: `email`, `firm` (or equivalent company name), `geo` (state at minimum). Recommended: `naics` or `industry`, `size_band`, `contact_name`, `role`, `consent_status`. Missing `naics`/`industry` will degrade sector-pack matching — the engine will fall back to general-language outreach.

---

## Compliance requirements (same five checks — buyer's data, buyer's responsibility)

The same five compliance checks that apply to DDDB contacts apply here. The buyer attests that:

1. **Provenance:** they know how and when each contact was collected.
2. **CAN-SPAM:** sending infrastructure identifies the sender, includes a physical address, and includes a working opt-out mechanism. Opt-outs will be honored within 10 business days.
3. **Suppression:** the buyer cross-references their suppression list before each run. Any contact on the suppression list is excluded.
4. **Consent status:** contacts with `consent_status = 'none'` are flagged for the buyer's own review before queuing. The engine will not auto-queue them.
5. **Jurisdiction:** the buyer has reviewed applicable laws for their target geographies.

The engine enforces the suppression check mechanically (cross-reference against the buyer's suppression list if provided). The buyer is responsible for the legal sufficiency of the other checks on their own data.

---

## CRM connector (optional)

If the buyer connects a CRM (HubSpot, Salesforce, etc.) instead of uploading a CSV, configure the connector in `config/routing.md` with the CRM's tool name and the field mapping to the normalized record schema. The adapter reads from the configured CRM tool; the same compliance checks apply.

---

## Legal notice

**The buyer is solely responsible for the legal sufficiency of their own contact list.** The plugin applies mechanical suppression and consent-status checks; it does not audit the buyer's data collection practices. Consult qualified legal counsel before contacting any cold prospect.
