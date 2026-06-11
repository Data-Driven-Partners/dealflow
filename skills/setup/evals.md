# setup — eval set (v1)

Merge gate for this skill. Two parts: **trigger activation** (fires on onboarding/config intent, not on prospecting intent) and **binary outcome checks** (deterministic checks against config/ file state).

`ambiguous` rows document the boundary and are excluded from the hard gate.

Activation threshold: >= 80% on combined should/should_not rows.

## Part 1 — trigger activation prompts

| kind | prompt | why |
|---|---|---|
| should | "set up the plugin" | Canonical onboarding trigger. |
| should | "how do I get started?" | First-run intent. |
| should | "fill in my config" | Direct config intent. |
| should | "configure dealmaker" | Direct config intent. |
| should | "onboard me" | Onboarding verb. |
| should | "what do I need to configure before prospecting?" | Pre-run setup inquiry. |
| should | "my config is incomplete, help me fill it in" | Mid-setup redirect. |
| should | "set up my DDDB connection" | DDDB-specific config intent. |
| should_not | "farm my leads" | Prospecting intent — routes to prospect. |
| should_not | "prospect for HVAC contractors" | Prospecting intent — routes to prospect. |
| should_not | "what is in my config/voice.md?" | Read-only config inquiry — answer directly, no rewrite. |
| should_not | "draft an outreach email" | Drafting intent — routes to prospect. |
| should_not | "update my proof library" | Proof-library task — routes to proof-library. |
| ambiguous | "check my config" | Could be read-only audit or active setup — document, don't gate. |
| ambiguous | "is my setup complete?" | Audit query that may or may not lead to fills — document, don't gate. |

Activation math: 8 should + 5 should_not = 13 decisive rows. Threshold: >= 80% (at most 2 wrong).

## Part 2 — binary outcome checks

Run after a setup session. A version fails the gate if any check regresses.

**Check A — all required config files are non-placeholder after a completed setup run.**
Manual review: open each config file after a setup session and confirm no `[PLACEHOLDER]` values remain in required fields. Any surviving placeholder = fail.

**Check B — attestation is complete before setup declares success.**
Manual review: confirm `config/dddb.md` contains all five compliance points confirmed with a date before the skill outputs "Setup complete." Declaring complete without attestation = hard fail.

**Check C — no secrets committed.**
Grep the config/ directory for patterns matching API keys, tokens, or passwords (e.g., strings of 20+ random alphanumeric characters, bearer tokens). Any secret committed = hard fail.
```bash
grep -rE '[A-Za-z0-9_\-]{32,}' config/ | grep -v PLACEHOLDER | grep -v '#'
# Any non-placeholder, non-comment long token = review immediately.
```

**Check D — prospect correctly blocks on missing attestation.**
Trigger `prospect` with `config/dddb.md` missing the attestation date. Confirm it halts and redirects to `setup`. If prospect proceeds without attestation = hard fail.
