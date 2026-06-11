---
name: setup
description: Onboards a new buyer by walking them through filling the config/ templates — company, products/fit-criteria, voice, proof sources, and DDDB connection with compliance attestation. Enforces the hard rule that prospect will not run a cold or DDDB source until config/dddb.md attestation is complete. Fire when: "set up the plugin", "configure dealmaker", "onboard", "first run", "fill in my config". Do NOT use when config/ is already complete and a buyer is ready to prospect.
metadata:
  version: 1.0.0
  author: Data Driven Partners
allowed-tools: [Read, Write]
---

# Setup — onboarding and configuration

Not a prospecting skill — a foundation. Before any prospect run can touch a cold or DDDB source, this skill ensures the buyer has filled every required config file and completed the compliance attestation. Incomplete config means the engine runs blind; missing attestation means it cannot legally reach cold contacts.

## The hard rule

`prospect` will NOT run a cold source (DDDB or BYO list) until `config/dddb.md` contains a completed compliance attestation (all five boxes checked, date recorded). This is enforced by `prospect` at step 0 — if attestation is absent, it halts and redirects to `setup`. There are no exceptions and no overrides.

## Triggers

**Fire this skill when …**
- the buyer says "set up the plugin", "configure dealmaker", "onboard", "first run", "fill in my config", "what do I need to configure", or "how do I get started".
- `prospect` detects missing or incomplete config and redirects here.
- any config file contains unfilled `[PLACEHOLDER]` blocks.

**Do NOT fire when …**
- config/ is already complete and attested — send the buyer to `prospect` instead.
- the buyer is asking about an existing config value (read-only inquiry — answer directly, no rewrite).

## The loop — do these in order

**1. Audit config/ for completeness.**
Read each config template in order:
- `config/company.md` — is the company name, description, and contact info filled?
- `config/products.md` — are offerings, fit criteria, and rate/price language filled? Any `[PLACEHOLDER]` remaining?
- `config/voice.md` — are voice rules and register(s) specified?
- `config/proof-sources.md` — is at least one proof source listed?
- `config/routing.md` — optional; note if blank but do not block.
- `config/dddb.md` — is the connection pointer filled AND is the five-point compliance attestation complete?

For each incomplete file, list the specific fields still showing `[PLACEHOLDER]` or blank.

**2. Walk the buyer through each gap, one file at a time.**
Do not overwhelm — present one config file at a time. Ask the buyer for the specific values needed. Write their answers into the config file immediately (no deferred batch write). Confirm each write before proceeding to the next file.

**3. Compliance attestation — config/dddb.md.**
This is the most important step. Read the five-point checklist in `config/dddb.md` aloud. Ask the buyer to confirm each point explicitly:

1. Provenance — "I have verified that every contact pulled from DDDB has a documented provenance record (where, when, and how collected)."
2. CAN-SPAM — "Every outreach email will identify the sender, include a physical mailing address, and include a working opt-out mechanism. Opt-outs will be honored within 10 business days."
3. Suppression — "I will maintain a suppression list and cross-check it before every batch. Anyone on the suppression list will be excluded unconditionally."
4. Consent status — "I understand that contacts with consent_status = 'none' will not be auto-queued. I will apply my own legal review before contacting them."
5. Jurisdiction — "I have reviewed my target geographies for CPRA, GDPR, CASL, and other applicable requirements, or I will obtain legal review before contacting contacts in those jurisdictions."

Write the buyer's confirmation and the date of attestation into `config/dddb.md`. **Without this step, prospect will not run a cold source.**

**4. Write the DDDB connection pointer.**
The buyer provides their DDDB API endpoint, credential reference (environment variable name — never the secret itself), and any filter defaults (default NAICS, size band, geo). Write these into `config/dddb.md` under the "Connection" section. Note clearly: never commit the actual API key or secret to any file.

**5. Confirm completion.**
Re-audit all config files. If all required fields are filled and the attestation is dated, confirm: "Setup complete. You can now run `prospect`." If any gap remains, loop back to step 2.

## Safeguards

- **Write, don't invent.** Write only what the buyer provides. Never fill in placeholder values with guesses or defaults that look like real configuration.
- **No secrets in files.** If the buyer pastes an API key, remind them to use an environment variable reference instead and write only the variable name.
- **Attestation is binary.** Either all five compliance points are confirmed with a date, or attestation is not complete. Partial attestation does not unlock `prospect`.

## Output

Updated config/ files + a confirmation line: "Setup complete: config/company.md, config/products.md, config/voice.md, config/proof-sources.md, config/dddb.md (attested [date]) — ready to run `prospect`."
Or, if incomplete: "Still needed: [list of specific gaps]. Resume `setup` to continue."
