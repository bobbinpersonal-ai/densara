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
- Bundle discount: automatic "Buy X, Get Y" discount (gid://shopify/DiscountAutomaticNode/1534167449817), "Fiber + Hold Bundle — 20% off Spray" — buy Sevich Hair Building Fibers, get Bunee FiberHold Spray 20% off. Bunee spray price is $23.00 standalone / $18.40 in the bundle (~$7.40 profit over ~$11 cost). Owner activated this one already (startsAt moved to 2026-07-25) — confirmed **ACTIVE** as of 2026-07-26.
- Lesson: `discountAutomaticDeactivate` sets an `endsAt` timestamp (marks it EXPIRED), which reads confusingly for a discount that never actually launched. Better pattern for a "draft" discount: set `startsAt` far in the future (with `endsAt: null`) for a clean SCHEDULED status instead.
- Draft theme "Helio - Copy content edits" (gid://shopify/OnlineStoreTheme/161842594009) holds rewritten, gender-inclusive homepage copy — NOT published. Live "Helio" theme is untouched. Check with owner before publishing.
- Blog "News" (gid://shopify/Blog/105200615641): all articles are published now, 25 total (author "DENSARA"), covering causes, personas (age/postpartum/fitness/hairline), how-to/ingredients, wig/hair-system anxiety angle, beard/brow angles, all with real product photos as featured images (no image generation available). All hair-loss-education content flagged as informational only, non-medical-advice, pointing to a dermatologist for diagnosis. 0 drafts pending as of 2026-07-26.
- Catalog expanded to 7 products via further DSers/CJ Dropshipping imports, spanning 3 areas: scalp (Sevich Fibers, Bunee Spray, Hoegoa Scalp Serum, Densara Derma Roller), beard (East Moon Beard Growth Oil, shares Derma Roller), brow/lash (Densara Eyebrow & Lash Fiber Filler, EELHOE Eyelash Serum). Same lesson repeated: always check real photos before branding — Hoegoa/East Moon/EELHOE are real third-party brands on otherwise DENSARA-vendor imports; Derma Roller and Brow/Lash Filler packaging is genuinely blank, safe under DENSARA.
- Strategic framing in use: 3 areas (scalp/beard/brow) × 2 time-horizons (instant fiber fix vs. long-term serum/oil/tool) as the bundle and content organizing structure. Sevich Fibers is being repositioned as usable on beard/brows too, not just scalp (same static-cling mechanism).
- 5 bundle discounts total now exist. Fibers→Spray (original, gid://shopify/DiscountAutomaticNode/1534167449817) is **ACTIVE**. The other 4 — Fibers→Beard Oil (1534589862105), Fibers→Scalp Serum (1534590058713), Beard Oil→Derma Roller (1534589927641), Brow/Lash Filler→Eyelash Serum (1534590025945) — are still **SCHEDULED** (startsAt 2099), same for the WELCOME10 code (gid://shopify/DiscountCodeNode/1534595072217) and the "Buy 2+ Fibers — Save 10%" volume discount (gid://shopify/DiscountAutomaticNode/1534595956953). Fibers anchors 3 bundles simultaneously.
- Eyebrow & Lash Fiber Filler (gid://shopify/Product/9821963256025) variant photo mapping: Shopify's own import auto-links each variant to its own photo (`variant.image`) — no manual linking needed. 24 supplier photos landed 2026-07-26 (rose gold, silver, gold, black, blue, pink, rhinestone, etc.), each already correctly assigned to its variant. Did not rename the confusing variant titles ("Set1", "Style 10", etc.) since 2 variants (Style 10 / Style 12) look like the same silver tube with no visible distinguishing detail — renaming risks mislabeling. Product actually has 33 total variants per the API (only ~10 have a distinct photo) — not investigated further, not launch-blocking.
- Launch-readiness audit (2026-07-26): all 7 products ACTIVE; live theme is "Helio - Copy content edits" (role MAIN); all 25 blog articles published. Vendor field on Sevich Hair Building Fibers was still "My Store" (generic import default) — fixed to "DENSARA". Only remaining pre-launch items: (1) activate the 4 scheduled bundle discounts + WELCOME10 + volume discount (all currently SCHEDULED by design, awaiting explicit owner go-ahead per safety rule 3/4), (2) remove storefront password protection (Online Store → Preferences — not exposed via Admin API, owner must do this manually).
