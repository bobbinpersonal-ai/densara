# Densara — Shopify Store Project

## About
- Store: Densara (Shopify)
- Product line: Sevich hair building fibers, for thinning hair
- Owner: beginner e-commerce operator — assume no prior Shopify/dev background, explain steps plainly
- This repo is the working directory for Claude Code sessions managing the store (docs, drafts, scripts, audit logs). It is not the Shopify theme repo itself unless/until we pull theme code into it.

## Session objectives (current)
1. Launch a new product
2. Set up a product bundle with a 20% discount offer
3. Launch a high-converting listicle landing page
4. Analyze recent sales data to find the highest-performing products
5. Audit the site's SEO and implement fixes

## Tooling status
- **Network access**: environment updated 2026-07-24 to Custom access including `shopify.dev`, `*.myshopify.com`, `admin.shopify.com`, `accounts.shopify.com` (plus default package-manager domains). Confirmed reachable.
- **Shopify AI Toolkit**: installed 2026-07-24 (`shopify-plugin@shopify-ai-toolkit`, v1.5.3) via `claude plugin marketplace add Shopify/shopify-ai-toolkit` + `claude plugin install shopify-plugin@shopify-ai-toolkit`. It's 20 skills (shopify-admin, shopify-liquid, shopify-onboarding-merchant, shopify-use-shopify-cli, etc.) plus 2 harness-only hooks — not raw MCP tools. It uses the `shopify` CLI / Admin GraphQL API under the hood.
  ⚠️ This toolkit mutates the LIVE store directly — no built-in draft mode, no undo. All safety rules below exist specifically to compensate for that.
  ⚠️ Skill scripts send usage telemetry (queries, code, model/client IDs) to shopify.dev **by default**. We've set `OPT_OUT_INSTRUMENTATION=true` in `.claude/settings.json` to disable this.
  Declared in this repo's `.claude/settings.json` (`enabledPlugins` + `extraKnownMarketplaces`) so it persists across sessions. **Plugins installed mid-session don't load their skills until a new session starts** — confirmed by testing.
- **shopify-dev-mcp** (read-only docs/schema helper, separate from the toolkit above): not installed — toolkit's `shopify-dev` skill likely covers this need already.
- **Admin API credentials**: not yet configured. Needed: a custom app in Shopify admin with Admin API access token, stored as an environment variable (never committed). See setup checklist below.

## Always-do safety rules
1. **Plan before mutating.** For every task that will create/update/delete a product, discount, theme file, or page, write out the plan (what will change, why, expected result) before making the call. Do not skip this even for "small" changes.
2. **Never edit the Live theme directly.** Always duplicate/create a draft (unpublished) theme, make changes there, and let the owner preview and manually publish. Never call a theme-publish action without explicit confirmation.
3. **Products and discounts start in Draft/Hidden/Inactive status.** Never set a new product, variant, or discount to Active/Published as part of its creation — that is a separate, explicitly-confirmed follow-up step.
4. **Confirm before anything irreversible or live-facing**, since the Shopify AI Toolkit itself has no undo. Treat every mutating tool call as one-way.
5. **Log every API call and file change to PROGRESS.md** as it happens (see that file for format).
6. **Never commit secrets.** Shopify Admin API access tokens, custom app credentials, etc. go in environment variables / local `.env` (gitignored), never in code or committed files.
7. Explain things in plain language — the owner is a beginner. Avoid unexplained jargon.

## Store schema notes (learned over time)
_(empty — will be filled in as we learn specifics about Densara's catalog, plan tier, apps installed, etc.)_
