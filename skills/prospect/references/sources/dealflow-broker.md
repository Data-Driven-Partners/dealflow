# Source adapter: dealflow-broker

**Adapter ID:** `dealflow-broker`
**Mode:** hosted (public product)
**Status:** primary source for customers using the hosted Dealflow product.
Use this adapter when the customer has a Dealflow account and has connected it
during plugin setup.  For self-host / direct-Supabase mode, see
`references/sources/dddb.md`.

---

## Purpose

In hosted mode the plugin does **not** hold any DDDB or Supabase credentials.
Instead it calls the Dealflow broker (`POST {DEALFLOW_BROKER_URL}/prospects`)
with the customer's Dealflow identity token.  The broker:

1. Verifies the token (Supabase Auth JWT).
2. Checks the customer's tier and daily quota.
3. Queries DDDB (server-side service-role access — the plugin never sees the
   DDDB connection string or key).
4. Optionally enriches records via Blitz (server-side; key stays on server).
5. Returns only the normalized prospect slice the customer is entitled to.

The plugin's only credential is the customer's own Dealflow identity JWT
(obtained when they signed in during setup).  It never touches DDDB or Blitz
directly.

---

## Call contract

```
POST {DEALFLOW_BROKER_URL}/prospects
Authorization: Bearer <dealflow-identity-token>
Content-Type: application/json
X-Idempotency-Key: <unique-string-per-pull>   ← prevents double-spend on retry

{
  "strategy": "<strategy_id>",   // "ideal_customer" | "competitor_employees" | "default"
  "n":        10,                 // number of prospects (1–100; server enforces cap)
  "filters":  {                   // optional; strategy-dependent
    "industry": "<string>",
    "title":    "<string>",
    "location": "<string>",
    "company":  "<string>"
  }
}
```

**Credentials the plugin sends:** only the `Authorization: Bearer <jwt>` header.
No DB keys, no Blitz key.  The `DEALFLOW_BROKER_URL` is set by the customer
during plugin setup (it is a URL, not a secret).

---

## Normalized prospect record (same schema across all adapters)

```json
{
  "prospect_id":    "<broker-assigned ID>",
  "firm":           "<company name — INTERNAL ONLY>",
  "contact_name":   "<full_name from response>",
  "role":           "<title>",
  "email":          "<email>",
  "naics":          null,
  "industry":       "<industry or null>",
  "size_band":      null,
  "geo":            "<location or null>",
  "source":         "dealflow-broker",
  "provenance_id":  "<id from broker response>",
  "consent_status": "opt-in",
  "collected_at":   null
}
```

The broker response body is:

```json
{
  "prospects": [
    {
      "id":           "<string>",
      "first_name":   "<string>",
      "last_name":    "<string>",
      "full_name":    "<string>",
      "company":      "<string>",
      "title":        "<string>",
      "email":        "<string>",
      "linkedin_url": "<string | null>",
      "phone":        "<string | null>",
      "industry":     "<string | null>",
      "location":     "<string | null>",
      "seniority":    "<string | null>",
      "department":   "<string | null>"
    }
  ],
  "meta": {
    "returned":      10,
    "used_today":    10,
    "cap":           50,
    "flagged_count": 2
  }
}
```

Map broker response fields → normalized record schema:

| Normalized field   | Broker response field     |
|--------------------|---------------------------|
| `prospect_id`      | `id`                      |
| `firm`             | `company`                 |
| `contact_name`     | `full_name`               |
| `role`             | `title`                   |
| `email`            | `email`                   |
| `industry`         | `industry`                |
| `geo`              | `location`                |
| `source`           | `"dealflow-broker"` (literal) |
| `consent_status`   | `"opt-in"` (literal — broker only returns compliant records) |
| all others         | `null` if absent          |

Records returned by the broker have already passed the server-side compliance
gate (provenance + consent + suppression).  The plugin must still run its own
qualification gate (`references/qualification-gate.md`) — that gate is about
fit, not data compliance.

---

## Error responses

| HTTP | `error` field         | Plugin action |
|------|-----------------------|---------------|
| 401  | `invalid_token`       | Prompt customer to reconnect their Dealflow account |
| 402  | `quota_exhausted`     | Surface `upgrade_url` to customer; do not retry |
| 403  | `profile_not_found`   | Prompt customer to complete Dealflow account setup |
| 500  | `db_error` / other    | Log and surface a generic "try again" message |

**402 quota_exhausted body:**

```json
{
  "error":       "quota_exhausted",
  "used":        10,
  "cap":         10,
  "credits":     0,
  "upgrade_url": "https://app.dealflow.io/upgrade"
}
```

When the plugin receives a 402, it must surface the `upgrade_url` to the
customer and halt the prospecting run.  Do not attempt to pull fewer prospects
to work around the quota — the server enforces it.

---

## Contrast with `dddb.md` (self-host / direct mode)

| | Hosted (this adapter) | Self-host (`dddb.md`) |
|---|---|---|
| Credentials in plugin | Dealflow JWT (identity only) | Supabase service-role key + connection string |
| DDDB access | Broker queries on your behalf | Plugin queries directly |
| Blitz enrichment | Broker-side, automatic | Not available |
| Quota enforcement | Broker-enforced | Operator-configured |
| Compliance gate | Broker + plugin | Plugin only |
| Setup complexity | Sign in once | Operator provisions Supabase |

---

## Setup (customer-facing)

1. The customer creates a Dealflow account at `https://app.dealflow.io`.
2. During plugin setup, they click "Connect Dealflow" → OAuth sign-in
   (Google or Microsoft via Supabase Auth).
3. The plugin stores the Dealflow access token + refresh token in its secure
   credential store.
4. Set `DEALFLOW_BROKER_URL` in the plugin environment to the broker base URL
   (provided in the Dealflow onboarding email).  This is a URL, not a secret.
5. The plugin is now ready.  No DB keys or Blitz keys are needed.
