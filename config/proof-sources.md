# Proof sources configuration

Fill in the sources the `proof-library` skill uses to build and refresh proof points. Two kinds of source:

- **OURS** — your own closed or completed transactions (tagged `source: OURS` in the library). These are the only entries that may be phrased "we closed" in outreach.
- **MARKET** — public datasets or industry reports (tagged `source: MARKET`). These show that deals of a given shape happen regularly — never "we closed."

---

## Proof library file path
[PATH TO YOUR PROOF LIBRARY FILE — e.g. "knowledge/proof-library.md"]
Default if left blank: `knowledge/proof-library.md`

---

## OURS sources (your own closed deals)

### Source 1: [CRM or deal tracker name]
Type: OURS
Description: [e.g. "Closed-won deals from HubSpot, exported quarterly"]
Access: [e.g. "HubSpot MCP connector" or "CSV export at path: knowledge/ours-deals.csv"]
Anonymization rules: [e.g. "Round amounts to nearest $50k band. Remove business names and owner names. Keep NAICS, size band, product type, region, term, and approximate close year."]
Last refreshed: [DATE OR LEAVE BLANK]

---

## MARKET sources (public data)

### Source 1: [DATA SOURCE NAME — e.g. "SBA 7(a) FOIA dataset"]
Type: MARKET
URL: [e.g. "https://data.sba.gov/en/dataset/7-a-504-foia"]
Covers: [e.g. "All SBA 7(a) and 504 loan approvals, FY1991–present. Fields: NAICS, approval amount, term, fiscal year, state, business type, jobs supported."]
How to use: [e.g. "Filter by NAICS and size band. Aggregate into patterns — never cite individual named rows. Ground rate language in config/products.md."]
Applicable sectors: [e.g. "SBA small business lending (sba-smallbiz-lending sector pack)"]
Last refreshed: [DATE OR LEAVE BLANK]

### Source 2: [ADD ADDITIONAL PUBLIC SOURCE IF NEEDED]
Type: MARKET
URL: [URL]
Covers: [DESCRIPTION]
How to use: [HOW TO AGGREGATE AND CITE]
Applicable sectors: [WHICH SECTORS]

---

## Notes

- Every MARKET source must be publicly accessible and citable. Do not include proprietary data in the MARKET category.
- OURS entries must be anonymized before being written to the library. The `proof-library` skill enforces this, but the buyer is responsible for confirming the anonymization is complete before any entry is used in outreach.
- The `proof-library` skill reads this file to know where to pull from. Keep source URLs current — SBA and other public datasets update their file URLs quarterly.
