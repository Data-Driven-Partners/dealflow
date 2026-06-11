# Routing configuration (optional)

Fill in your team's deal routing rules. The `prospect` skill reads this at handoff (step 11) to determine who should own a replied-to prospect. Leave sections blank if you do not have dedicated routing — the engine will surface replies without a specific owner.

---

## Default owner (if no routing rule matches)
Name: [DEFAULT OWNER NAME OR LEAVE BLANK]
Email: [DEFAULT OWNER EMAIL OR LEAVE BLANK]

---

## Routing rules (add as many as needed)

### Rule 1: [RULE NAME — e.g. "SBA-eligible prospects"]
Condition: [WHEN THIS RULE APPLIES — e.g. "Prospect NAICS is SBA-eligible and product fit is SBA 7(a)"]
Owner: [NAME]
Email: [EMAIL]
Note: [OPTIONAL NOTE — e.g. "CC the COO on deals over $1M"]

### Rule 2: [RULE NAME — e.g. "Non-SBA or conventional deals"]
Condition: [WHEN THIS RULE APPLIES]
Owner: [NAME]
Email: [EMAIL]
Note: [OPTIONAL NOTE]

### Rule 3: [ADD MORE RULES AS NEEDED]

---

## Handoff note format
When `prospect` hands off a replied-to lead, it writes a one-line note. Suggested format:
"[Strategy that worked] / [Angle] / [Lead score if tracked] / [Product fit]. Handoff to [owner]."

---

## Notes
- This file is optional. If left empty, the engine does not attempt to route — it surfaces the reply for the buyer to handle.
- Do not put email credentials or API keys here. This file is for routing logic only.
