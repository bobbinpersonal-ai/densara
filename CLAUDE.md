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
- **Admin API credentials**: CONFIGURED 2026-07-24. Custom-app static tokens are deprecated for new apps as of Jan 2026; the current path is Shopify CLI's `shopify store auth --store <domain> --scopes <...>` (OAuth, no custom app/backend needed). Authenticated as bobbin.dahal@mail.com. Re-run `store auth` if the session token expires or more scopes are needed.
- **⚠️ Correct store domain is `fa8ihd-6j.myshopify.com`, NOT `densara.myshopify.com`.** "Densara" is the store's display name only — `densara.myshopify.com` does not exist (confirmed 404). Always use `fa8ihd-6j.myshopify.com` for CLI/API calls.
- **Store plan**: Basic.
- **⚠️ Node/proxy quirk**: Shopify CLI 4.x uses Node's native `fetch`, which does not honor `HTTPS_PROXY` unless `NODE_USE_ENV_PROXY=1` is set (Node ≥ 22.21). Without it, every CLI call that reaches Shopify's servers fails with a misleading `403 Host not in allowlist` error that looks like a network-policy block but isn't. Set permanently in this repo's `.claude/settings.json` under `env`.

## Always-do safety rules
1. **Plan before mutating.** For every task that will create/update/delete a product, discount, theme file, or page, write out the plan (what will change, why, expected result) before making the call. Do not skip this even for "small" changes.
2. **Never edit the Live theme directly.** Always duplicate/create a draft (unpublished) theme, make changes there, and let the owner preview and manually publish. Never call a theme-publish action without explicit confirmation.
3. **Products and discounts start in Draft/Hidden/Inactive status.** Never set a new product, variant, or discount to Active/Published as part of its creation — that is a separate, explicitly-confirmed follow-up step.
4. **Confirm before anything irreversible or live-facing**, since the Shopify AI Toolkit itself has no undo. Treat every mutating tool call as one-way.
5. **Log every API call and file change to PROGRESS.md** as it happens (see that file for format).
6. **Never commit secrets.** Shopify Admin API access tokens, custom app credentials, etc. go in environment variables / local `.env` (gitignored), never in code or committed files.
7. Explain things in plain language — the owner is a beginner. Avoid unexplained jargon.

## Store schema notes (learned over time)
- Existing product: "Sevich Hair Building Fibers" (gid://shopify/Product/9820803989721), status ACTIVE. Single option named "format" combining shade+size, e.g. "Black 25g". Real shade lineup (10 total): Black, Dark Brown, Medium Brown, Light Brown, Auburn, Golden Blonde, Medium Blonde, Light Blonde, Grey, White. Sizes: 12g (White only), 25g (all other shades). There is also a standalone "Applicator" variant (accessory, not a shade/size). No 100g size exists despite what old copy claimed — verify against live variants before writing any size/shade claims.
- Storefront was password-protected as of 2026-07-25 (pre-launch). Check before assuming any page is publicly visible/indexable.
- No custom domain connected as of 2026-07-25 — still on `fa8ihd-6j.myshopify.com`.
- Live theme: "Helio" (stock Shopify 2026 theme). Has solid built-in technical SEO (title tags, meta description, canonical, Open Graph, Twitter Card) via `snippets/meta-tags.liquid` — don't assume it needs fixing without checking first.
- Image alt text: use the `fileUpdate` mutation with `MediaImage` GIDs (get these via `product.media`, NOT `product.images` which returns `ProductImage` GIDs that don't work with `fileUpdate`).
- `cdn.shopify.com` needed adding to the environment's allowed domains separately from `*.myshopify.com` in order to download/view product images.
- Second product: DSers/CJ Dropshipping import created a SEPARATE product ("Super Strong Hair Fiber Modeling Spray") instead of mapping to our custom draft — the import is the one with real fulfillment/inventory/photos. Consolidated by moving branding onto the real supplier product and deleting the orphaned draft (gid://shopify/Product/9821429367001, deleted 2026-07-25).
- **Real bottle is branded "Bunee" (third-party brand), not "Sevich"** — confirmed by viewing actual product photos. Owner chose to sell it honestly under the Bunee name rather than mismatch branding vs. the physical item. Current listing: "Bunee FiberHold Spray" (gid://shopify/Product/9821441392857), vendor "Bunee", $22.00, ACTIVE status (was already Active from the import; not changed), SKU CJJT284821301AZ (CJ Dropshipping), 5,404 in stock, 5 real product images with alt text set.
- Lesson: always check the ACTUAL product photo before applying private-label branding to a dropshipped item — don't assume packaging is blank just because the store's working title is generic.
- Bundle discount: automatic "Buy X, Get Y" discount (gid://shopify/DiscountAutomaticNode/1534167449817), "Fiber + Hold Bundle — 20% off Spray" — buy Sevich Hair Building Fibers, get Bunee FiberHold Spray 20% off. Currently **SCHEDULED** (startsAt 2099-01-01, inactive) per draft-discount safety rule; owner activates by moving startsAt when ready to launch. Bunee spray price is $23.00 standalone / $18.40 in the bundle (~$7.40 profit over ~$11 cost).
- Lesson: `discountAutomaticDeactivate` sets an `endsAt` timestamp (marks it EXPIRED), which reads confusingly for a discount that never actually launched. Better pattern for a "draft" discount: set `startsAt` far in the future (with `endsAt: null`) for a clean SCHEDULED status instead.
- Draft theme "Helio - Copy content edits" (gid://shopify/OnlineStoreTheme/161842594009) holds rewritten, gender-inclusive homepage copy — NOT published. Live "Helio" theme is untouched. Check with owner before publishing.
- Blog "News" (gid://shopify/Blog/105200615641) now has 4 unpublished draft articles on hair-loss education (men's and women's causes separately, plus an immediate-vs-long-term piece). All flagged as informational only, non-medical-advice, pointing to a dermatologist for diagnosis.
