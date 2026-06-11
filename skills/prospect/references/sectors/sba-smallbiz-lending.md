# Sector pack: SBA Small Business Lending

**This is the flagship worked example sector pack.** It uses public SBA FOIA data (data.sba.gov) as the MARKET proof source. It is the one place in this engine where SBA-specific details appear. All NFF-specific buyer details (products, rates, routing) belong in `config/` — not here.

---

## Sector name
Small Business — SBA 7(a) and 504 Lending (commercial small-business lending to operating businesses via SBA guaranteed programs)

## NAICS prefixes
This sector pack is product-defined, not NAICS-defined. It applies to any operating business (broadly, NAICS 11–81 excluding SBA-ineligible industries) that:
- Has been in operation for 2+ years.
- Has annual revenue above the lender's minimum (configure in `config/products.md`).
- Is structured as a for-profit entity in the United States.
- Does not appear on SBA's ineligible business list (https://www.sba.gov/sites/default/files/2021-01/SBA_Ineligible_Businesses.pdf).

Sector pack is most applicable across NAICS 23x (construction/contractors), 44x–45x (retail), 48x–49x (transportation), 53x (real estate), 54x (professional services), 56x (administrative services), 61x (education), 62x (health care), 71x (arts/recreation), 72x (accommodation/food service), 81x (other services). Confirm SBA eligibility before applying to any specific NAICS.

## Typical size bands
- **$150k–$500k** — most common band for 7(a) working capital, equipment, and light expansion.
- **$500k–$2M** — common for owner-operator growth, acquisitions, commercial real estate light.
- **$50k–$150k** — smaller operators; SBA Express program is most relevant here.
- **$2M–$5M** — larger 7(a) or 504 (real estate/heavy equipment). 504 is the primary vehicle at the top of this band.

## Common products (SBA-specific — map to your config/products.md)

The products listed here are the SBA program types. Your `config/products.md` should list your specific offerings and rates. Map the programs to your product names:

- **Primary — SBA 7(a) term loan:** working capital, equipment, debt refinancing, business acquisition. The default SBA product for most small businesses. Most SBA lenders offer it; the buyer's product config determines their specific criteria and rates.
- **Secondary — SBA 7(a) Express:** faster underwriting (36-hr SBA response); loan limit $500k; higher rates; lower SBA guaranty. Good for urgency and smaller deals.
- **Secondary — SBA 504:** real estate and major equipment. Structured as bank (50%) + CDC (40%) + borrower equity (10%). 10-year and 20-year terms; fixed rate on CDC tranche.
- **Less common — SBA USDA B&I:** rural businesses; separate program; include only if the buyer has this product.
- **SBA-ineligible industries:** see https://www.sba.gov/sites/default/files/2021-01/SBA_Ineligible_Businesses.pdf. Do not route ineligible businesses to SBA products.

## Language and jargon to translate

| SBA term | How to use in outreach |
|---|---|
| "7(a)" | "SBA loan" or "SBA 7(a)" — most owners recognize "SBA loan" |
| "504" | "SBA 504" or "commercial real estate / equipment loan" — explain the structure briefly |
| "PG (personal guarantee)" | State plainly: "a personal guarantee is required from any 20%+ owner — this is standard for SBA loans" |
| "DSCR" | "debt service coverage" — or explain inline: "does your cash flow cover the monthly payment" |
| "SBA guarantee fee" | Explain that the fee is paid to the SBA; lender does not retain it |
| "subordination" | "other lenders agree to take second position" |
| "injection / equity injection" | "your down payment on the deal" |
| "collateral" | "business assets pledged as security; home lien is NOT required for loans under $500k" |

## Seasonal and timing patterns

- **Q1 (Jan–Mar):** post-holiday cash flow recovery; owners assess the year ahead. Good window for LOC setup and working capital conversations.
- **Q2 (Apr–Jun):** growth season for most industries; capital for hiring and expansion.
- **Q3 (Jul–Sep):** rate environment watch; SBA often announces program fee changes for the new fiscal year (Oct 1).
- **Q4 (Oct–Dec):** SBA fiscal year starts October 1; new fee structures, Express limits, and program changes often take effect. Strong timing hook.
- **SBA fiscal year:** October 1 – September 30. New-year program changes are a recurring timing signal.
- **Rate environment:** SBA 7(a) variable rates are tied to prime rate (Wall Street Journal Prime). Fed rate decisions are recurring timing signals — flag significant moves.

## Qualification nuances

- **Time-in-business (TIB):** 2 years is the standard SBA floor. Some lenders require more; check `config/products.md`. Businesses at 18–23 months TIB are near-term qualifiable — note the path.
- **Revenue floor:** configure in `config/products.md`. The SBA does not set a minimum; lenders do. A common floor is ~$100k–$200k annual revenue depending on the lender.
- **SBA ineligible industries:** check the SBA ineligible list before routing. Common ineligible: passive investment, lending, gambling, lobbying, religious organizations. If ineligible for SBA AND no non-SBA product fits → real no.
- **Business structure:** must be a for-profit U.S. entity. Sole proprietors may apply; check lender requirements.
- **Credit:** SBA does not set a minimum credit score; lenders typically require 650+ personal credit for the primary owner. A recent bankruptcy or open tax lien is often a hard disqualifier unless resolved.
- **Product-mismatch to reframe:** if SBA doesn't fit (too early for TIB, below revenue floor), a conventional term loan, LOC, or equipment financing may still fit — this is a `product-reframe` candidate, not a real no.

## Proof-point pointers

**MARKET source (public SBA FOIA data):**
The SBA FOIA dataset is the primary public source for comparable transaction patterns in this sector.
- Dataset: https://data.sba.gov/en/dataset/7-a-504-foia
- 7(a) current file (resolve the live filename from the dataset page each run — the `asof` date changes quarterly): https://data.sba.gov/dataset/0ff8e8e9-b967-4f4e-987c-6ac78c575087/resource/d67d3ccb-2002-4134-a288-481b51cd3479/download/foia-7a-fy2020-present-as-of-251231.csv
- 504 current file: likewise from the dataset page.
- Useful fields: `NaicsCode`, `NaicsDescription`, `GrossApproval`, `ApprovalFiscalYear`, `TermInMonths`, `BusinessType`, `JobsSupported`, `subprogram`.
- Do NOT store `BorrName` — anonymize all outputs. Never link a deal amount to a named borrower.
- Ground rate language in `config/products.md` (the SBA file shows deal shapes and amounts, not current rates).

**How to use this data for ib-comp-book:**
Filter by NAICS (match the prospect's 3-digit prefix), size band, and recent fiscal years. Aggregate into patterns — never cite individual named deals. Present as: "[Industry] businesses in the [region] received SBA 7(a) loans in the [size band] range routinely in FY[year range]." Always cite the data source.

**MARKET ≠ OURS:** Data from data.sba.gov covers all SBA lenders — not the buyer's own closed deals. Never phrase a FOIA-sourced pattern as "we closed this." Only deals in the buyer's own proof library tagged `source: OURS` may use "we closed."

## Public data sources for this sector

- Source: SBA 7(a) & 504 FOIA dataset
  URL: https://data.sba.gov/en/dataset/7-a-504-foia
  Covers: All SBA 7(a) and 504 loan approvals, FY1991–present (current quarterly updates). Fields: NAICS, approval amount, term, fiscal year, state, business type, jobs supported.
  Use as: MARKET proof (never "we closed")

- Source: SBA Newsroom / Press Releases
  URL: https://www.sba.gov/about-sba/sba-newsroom
  Covers: Program changes, fee announcements, limit adjustments, new initiatives.
  Use as: Timing signals (cite the specific announcement)

- Source: Federal Reserve / Wall Street Journal Prime Rate
  URL: https://www.wsj.com/market-data/bonds/moneyrates
  Covers: Current prime rate (SBA 7(a) variable rates are set at prime + spread).
  Use as: Rate-environment timing signal

## Tone notes

Small business owners (the prospect in this sector) respond to:
- **Directness and specificity.** "Here's what a deal shaped like yours actually looks like" beats "we have great options."
- **Honesty about the process.** SBA has a reputation for being slow and bureaucratic. Acknowledge it; explain what makes the path faster or easier without overpromising.
- **The math.** A monthly payment figure and a payback period are worth more than any adjective. Show the math.
- **Peer credibility.** A comp from a business in their industry or region is more persuasive than a national statistic.
- **No pressure.** Owners are pitched constantly. An easy out and no fake urgency build trust faster than manufactured scarcity.

## Common objections in this sector

- **"SBA is too slow / too much paperwork"** → acknowledge the friction; distinguish fast-track paths (Express, streamlined lenders) from the full 7(a) process. "The slowness is real — but the rate is worth it if you qualify."
- **"I tried before and got denied"** → prior denial is not permanent. Ask what changed (time in business, revenue, credit). A prior SBA denial often reveals the path to a non-SBA product.
- **"I don't need capital right now"** → LOC reframe: "the cheapest time to set one up is when you don't need it — it costs nothing until you draw."
- **"The rate is too high"** → show the math vs alternatives (e.g. daily MCA debit converted to one monthly SBA payment). Rate objection is often a buying signal.
- **"I own a [specific ineligible business type]"** → if genuinely SBA-ineligible, acknowledge it honestly and route to a non-SBA product if available — do not push SBA to someone who can't use it.
