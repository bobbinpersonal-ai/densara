# Progress / Audit Log

Every API call and file change made during Densara Claude Code sessions is logged here, most recent first.

Format: `- [date/time] TYPE — description — result`
Types: `FILE`, `API-READ`, `API-MUTATE`, `PLAN`, `DECISION`

---

- [2026-07-24] FILE — Created `CLAUDE.md` with project conventions and safety rules.
- [2026-07-24] FILE — Created `PROGRESS.md` (this file).
- [2026-07-24] DECISION — Environment check: no `shopify-ai-toolkit` plugin or `shopify-dev-mcp` server installed in this session (confirmed via ListPlugins/ListConnectors, both empty).
- [2026-07-24] DECISION — Confirmed via direct network test that this cloud environment's outbound network policy blocks `shopify.dev` and `*.myshopify.com` (explicit policy-403 on both). Live Shopify operations are not possible from this session until network policy is changed or work moves to a local Claude Code session.
- [2026-07-24] DECISION — Owner chose to change this environment's network access level to Custom, adding Shopify domains to the allowlist, rather than moving work to a local machine. Waiting on owner to make the change in environment settings before proceeding with live-store objectives.
- [2026-07-24] DECISION — Owner confirmed network access settings saved. Re-tested connectivity: shopify.dev now returns HTTP 200, densara.myshopify.com resolves, no new proxy relay failures. Network access confirmed open.
- [2026-07-24] API-MUTATE (local CLI, not store) — Ran `claude plugin marketplace add Shopify/shopify-ai-toolkit` — cloned marketplace successfully.
- [2026-07-24] API-MUTATE (local CLI, not store) — Ran `claude plugin install shopify-plugin@shopify-ai-toolkit` — installed v1.5.3, scope: user.
- [2026-07-24] FILE — Created `.claude/settings.json` declaring the marketplace + enabled plugin, and setting `OPT_OUT_INSTRUMENTATION=true`, so the plugin persists into future sessions on this repo/branch and telemetry to shopify.dev is disabled by default.
- [2026-07-24] DECISION — Confirmed the toolkit's skills are not yet loaded in this running session (mid-session plugin installs require a fresh session start). No store data has been accessed or mutated yet. Still need: Shopify Admin API custom app + access token before any live-store action can happen.
- [2026-07-24] DECISION — Discovered custom-app static Admin API tokens (Settings → Apps → Develop apps) are deprecated for new apps as of Jan 1, 2026. Owner had instead created an app via the newer Dev Dashboard. Its Client ID/Secret and "App automation token" are for OAuth backends / CI-CD app deployment respectively — neither gives direct Admin API data access, so both were set aside unused.
- [2026-07-24] API-MUTATE (local CLI) — Installed `@shopify/cli` v4.5.2 globally via npm.
- [2026-07-24] DECISION — Found the correct mechanism: `shopify store auth --store <domain> --scopes <...>` performs a browser OAuth flow and stores a reusable session token, no custom app needed.
- [2026-07-24] DECISION — Discovered the store domain `densara.myshopify.com` used throughout earlier steps was wrong (404 — store doesn't exist). "Densara" is only the display name; the real store handle is `fa8ihd-6j.myshopify.com`, confirmed via the owner's screenshot of a working admin.shopify.com session.
- [2026-07-24] DECISION — Diagnosed and fixed a Node.js proxy quirk: Shopify CLI 4.x's native `fetch` ignores `HTTPS_PROXY` unless `NODE_USE_ENV_PROXY=1` is set, causing a misleading `403 Host not in allowlist` error on every real API call. Fixed by setting `NODE_USE_ENV_PROXY=1` in `.claude/settings.json` env block.
- [2026-07-24] API-READ (Admin GraphQL, read-only) — Ran `shopify store auth` (OAuth flow, owner approved in browser) then verified with `shopify store execute --query 'query { shop { name id myshopifyDomain plan { displayName } } }'`. Result: store name "DENSARA", plan "Basic", domain fa8ihd-6j.myshopify.com. **Authentication confirmed working end-to-end.**
- [2026-07-24] DECISION — Note: the CLI's stored auth token lives in this container's local config, not in the repo. It may not persist to a fresh cloud session/container — if `shopify store execute` reports not authenticated in a future session, re-run `shopify store auth` (see CLAUDE.md).
