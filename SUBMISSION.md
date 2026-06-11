# Getting Listed in the Anthropic Community Marketplace

This document covers the precise path to getting `dealflow` listed in the Anthropic Claude plugin community marketplace.

---

## Where submissions go

**Submissions are made through the form at:**
**https://clau.de/plugin-directory-submission**

Do NOT open a pull request directly to `anthropics/claude-plugins-community`. Direct PRs to that repo are auto-closed. The submission form is the only accepted path.

After approval, the plugin is mirrored nightly into `anthropics/claude-plugins-community` and becomes installable via [claude.ai/plugins](https://claude.ai/plugins).

---

## Prerequisites before submitting

Complete every item before clicking submit. Anthropic's review scans for these; an incomplete submission will be rejected or delayed.

### Repo

- [ ] The GitHub repo `data-driven-partners/dealflow` is **PUBLIC**.
- [ ] The default branch is clean and the files below are present at the repo root.

### Required files at root

- [ ] `.claude-plugin/marketplace.json` — valid JSON, no BOM, LF line endings, matches the schema documented in Anthropic's submission guide.
- [ ] `.claude-plugin/plugin.json` — valid JSON, no BOM, LF line endings, includes `interface` block with `displayName`, `shortDescription`, `longDescription`, `category`, `capabilities`, `websiteURL`, `privacyPolicyURL`, `termsOfServiceURL`, `defaultPrompt`, and `brandColor`.
- [ ] `LICENSE` — MIT (or another OSI-approved license). A `license` field in `marketplace.json` and `plugin.json` must match the file.
- [ ] `README.md` — complete, describes what the plugin does, includes install instructions for both Cowork and Code, and links to `PRODUCT-SPEC.md` for architecture detail.

### URLs — must be live before submitting

**The privacy policy and terms of service URLs referenced in `plugin.json` must resolve to real pages before you submit.** These are required by Anthropic's review.

- [ ] `https://datadrivenpartners.com/privacy` is live and contains a real privacy policy.
- [ ] `https://datadrivenpartners.com/terms` is live and contains real terms of service.

If these pages do not exist yet, create them before submitting. The review will test the URLs.

### Security scan

Anthropic runs automated security scanning on submitted repos. The plugin will be rejected if:

- [ ] No secrets, tokens, API keys, or credentials are committed anywhere in the repo. (Check with `grep -riE "ghp_|github_pat_|gho_|api[_-]?key|secret" .` — should return nothing but placeholder text.)
- [ ] No obfuscated code.
- [ ] No remote code execution (no `eval`, no fetching and running remote scripts).
- [ ] `allowed-tools` in each skill's SKILL.md front matter lists only the tools the skill actually uses — no over-claiming permissions.

### Plugin behavior

Anthropic reviewers test plugin behavior. Confirm:

- [ ] Every skill that sends messages uses **draft-not-send** — the user approves every send. This plugin satisfies this: `prospect` queues drafts; the approval queue artifact is the only send path.
- [ ] No skill sends automatically without user approval.
- [ ] The `setup` skill successfully walks a new user through configuration from zero.
- [ ] The compliance gate in `prospect` halts on missing attestation and redirects to `setup`.

---

## Submission checklist (operator: Tristan)

Complete in order:

1. [ ] Push the current repo state to `github.com/data-driven-partners/dealflow` and confirm it is public.
2. [ ] Publish `https://datadrivenpartners.com/privacy` (real privacy policy page).
3. [ ] Publish `https://datadrivenpartners.com/terms` (real terms of service page).
4. [ ] Run `python3 -m json.tool .claude-plugin/marketplace.json` and `python3 -m json.tool .claude-plugin/plugin.json` locally to confirm both are valid JSON.
5. [ ] Run the secrets grep: `grep -riE "ghp_|github_pat_|gho_|api[_-]?key|secret" .` and confirm no real secrets are present.
6. [ ] Install the plugin in Claude Code locally and run `setup` end-to-end to confirm it works.
7. [ ] Go to **https://clau.de/plugin-directory-submission** and complete the form.

---

## After approval

- Anthropic mirrors the repo nightly into `anthropics/claude-plugins-community`.
- The plugin appears at [claude.ai/plugins](https://claude.ai/plugins) and is installable via `claude plugin marketplace add data-driven-partners/dealflow`.
- To update the listing, push to the repo — the nightly mirror picks up changes automatically.
- To unpublish, contact Anthropic support or use the form linked in the marketplace developer docs.

---

## Notes

- The submission form asks for the GitHub repo URL, a short description, and contact information. Use `https://github.com/data-driven-partners/dealflow`, the short description from `marketplace.json`, and `tristan@datadrivenpartners.com`.
- If the review finds issues, Anthropic will email the contact address in `marketplace.json` with specific feedback. Correct and resubmit via the same form.
