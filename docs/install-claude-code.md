# Install: Claude Code

Install **Dealflow** from the Anthropic community marketplace via the Claude Code CLI.

## Prerequisites

- Claude Code installed and authenticated (`claude --version`)
- Git and a working shell

## Install

```bash
claude plugin marketplace add data-driven-partners/dealflow
claude plugin install dealflow
```

## First run

After installing, run the `setup` skill:

```
Set up my deal-maker prospecting toolkit.
```

The `setup` skill will walk you through filling in each file in `config/`:

- `config/company.md` — your company name, sender, physical address (required for CAN-SPAM)
- `config/products.md` — your offerings, fit criteria, and honest rate/price language
- `config/voice.md` — your voice rules and register(s)
- `config/proof-sources.md` — your proof sources (your closed deals and public data)
- `config/dddb.md` — optional: connected data provider endpoint and compliance attestation

Setup will not declare complete until the compliance attestation is dated (required if you plan to use a connected cold-contact source).

## After setup

Start prospecting:

```
Prospect for [industry] businesses in [geography].
```

Or work an existing list:

```
Work my dormant list and draft re-engagement outreach.
```

Review the approval queue artifact, edit any draft, and approve to send.

## Updating

```bash
claude plugin update dealflow
```

## Uninstalling

```bash
claude plugin uninstall dealflow
```
