# DDDB connection (Supabase) + compliance attestation

This configures the **DDDB contact source** — your contacts live in your **Supabase** instance — and holds the mandatory compliance attestation. The `prospect` skill checks this file before running any DDDB-sourced outreach. **Attestation must be complete before `prospect` will activate a cold source.**

**Never commit secrets.** Reference environment-variable names only — never paste a Supabase key into this file.

---

## Connection (Supabase)

- **Supabase project ref / URL:** `[e.g. https://<ref>.supabase.co]`
- **Service key env var name (the variable name, NOT the key):** `[e.g. SUPABASE_DDDB_KEY]`
- **Contacts table:** `[schema.table — e.g. dddb.contacts]`
- **Suppression table:** `[schema.table — e.g. dddb.suppression]`

### Column mapping (your table column → normalized prospect field)
Map your table's columns to the fields the engine expects. Any unmapped **required** field fails the gate.

| normalized field | your column | required |
|---|---|---|
| prospect_id | `[id]` | yes |
| firm | `[company]` | yes |
| contact_name | `[full_name]` | yes |
| role | `[title]` | |
| email | `[email]` | yes |
| naics | `[naics]` | |
| industry | `[industry]` | |
| size_band | `[size_band]` | |
| geo | `[state]` | |
| provenance_id | `[provenance_id]` | **yes (compliance)** |
| consent_status | `[consent_status]` | **yes (compliance)** |
| collected_at | `[collected_at]` | **yes (compliance)** |

### Default filters
- Default NAICS: `[e.g. 238, 722 — or blank]`
- Default size band: `[e.g. $150k–$2M — or blank]`
- Default geography: `[e.g. TX, FL, GA — or blank]`
- Default max contacts per run: `[e.g. 50]`

---

## Five-point compliance attestation (required before cold sourcing)

`prospect` enforces this mechanically: if the date below is blank or any item is unconfirmed, it halts cold sources and redirects to `setup`. Replace `[ ]` with `[x]` when confirmed.

- `[ ]` **1. Provenance** — every contact row has a non-null `provenance_id` (where/when/how collected); rows without one are never queued.
- `[ ]` **2. CAN-SPAM** — sends identify the sender, include a physical address (config/company.md) + working opt-out; opt-outs honored ≤10 business days.
- `[ ]` **3. Suppression** — the suppression table is current and cross-referenced on every pull; matches excluded unconditionally.
- `[ ]` **4. Consent** — `consent_status` is `opt-in` or DDDB-cleared; `none` rows are held for legal review, not auto-sent.
- `[ ]` **5. Jurisdiction** — CPRA/state data-broker + international (GDPR/CASL) obligations reviewed for the contacted regions.

**Attested by:** `[name]`  **Date:** `[YYYY-MM-DD]`  **Legal review on file:** `[link/ref]`

---

## Legal notice

Completing this attestation is **not** legal advice and does **not** guarantee compliance. The plugin enforces mechanical checks; it does not evaluate legal sufficiency in any jurisdiction. Obtain your own legal review before contacting cold prospects.
