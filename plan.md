# DealNestAI hardening and permanent website plan

## Goal
Deploy the existing DealNestAI application as a permanent managed website; remove the missing-item-link interruption for source-verified products; show only source-backed deal observations with an honest freshness and fallback state; secure administrator PIN rotation so the former `12345` code no longer authenticates; make Operations & Health distinguish partner website reachability from actual market-data availability; make Link Price Inspector resolve verified retailer short links to exact product pages without fabricated comparisons or timeout failures; and let admins securely manage multiple Gemini/Groq keys used by a reliable DealNest Intelligence Concierge.

## Implementation approach
- Preserve the uploaded React/Vite + Express monorepo, navigation, catalog, customer flows, and admin dashboard. Use the existing server entry point and container for a single-origin deployment.
- In Link Price Inspector, normalize case, trailing-slash, and tracking-query variants of a known Flipkart short-link path to its exact verified item page; parse Flipkart's `id="jsonLD"` product data and variant `pid`, and show the exact observed product/price. For other links, follow bounded redirects only across the supported retailer's hosts and require exact structured product identity before showing a price.
- Add exact Amazon India / Flipkart product-detail URLs for two verified matching products (iPhone 17 512 GB and Sony WH-1000XM5). Never resolve an unmatched product into a search, category, offers, or homepage URL; make an unmapped offer non-blocking and avoid claiming it has an exact item page.
- Poll only those known official product pages through bounded server-side requests and cached checks; display source, check time, price, and offer eligibility. If a marketplace blocks the request, parsing fails, or the data ages, label that fact and state the fallback. Public-page polling is not an authorized partner API, so do not mislabel it as one.
- Remove every hard-coded legacy administrator-code bypass. Rotate the current code to a custom PIN while retaining salted hashing and rate limits, then verify `12345` cannot log in.
- For development Preview only, if `ADMIN_ACCESS_CODE` is missing or malformed, hash an unexposed random verifier, invalidate existing admin sessions, and lock admin login so public pages still render without restoring any default PIN. Production must always refuse to start if the configured PIN is missing or not 8–12 digits; do not publish until production PIN configuration passes this check.
- Extend Operations & Health to show Amazon and Flipkart homepage response latency separately from item-price check status, plus the active deal-data source/fallback and last-check time.
- Build and deploy the existing Express-served Vite output on the managed website. Preserve the uploaded logo and one-page route manifest.
- Extend the existing Admin Dashboard with a Gemini/Groq key manager: add multiple labeled keys with provider-specific models, test them, and remove them. Keep both provider adapters always included in the Concierge failover chain; do not add a provider-disable toggle. Report a provider as active only after at least one key passes a test, and show `Key required` when credentials are missing. Restrict provider selection to Gemini and Groq—no arbitrary base URL—so admin input cannot turn the server into an SSRF proxy. All mutation/list/test routes require the existing `requireAdmin` authorization; return only safe metadata and a masked last-four identifier, never the key.
- Enable managed MySQL and add an additive, idempotent `ai_provider_keys` table. Store provider keys only as AES-256-GCM ciphertext with a unique nonce and authentication tag, using `DEALNEST_AI_KEY_ENCRYPTION_KEY` from a protected server secret; do not write plaintext keys to logs, files, responses, source control, or the browser. Keep schema startup lazy/fast so the unauthenticated health route remains available even if the database is temporarily unavailable.
- In `/api/ai/chat`, try every stored Gemini key in stable creation order, then every stored Groq key; on provider errors/timeouts continue to the next key. Use sequential failover rather than calling both on every message, avoiding duplicate API charges. Retain the existing server-environment keys as compatibility fallbacks, then the safe catalog-context reply. Grounded labels and citations are returned only when actual provider grounding sources exist; never invent live prices, coupons, inventory, or coverage.
- Preserve the current Admin Dashboard visual language and single-page route set. Add a dedicated AI API Keys panel with status, provider/model, masked key suffix, add/test/remove controls; the customer Concierge continues to use the existing authenticated chat route. Keep Gemini/Groq names in admin management only; customer-facing Concierge labels use neutral DealNest evidence wording, retain actual citations/source links and DealNest's own copyright, and do not present provider copyright/branding text.

## Evidence and operating constraints
Official Amazon India and Flipkart item pages were directly checked during implementation. Product price, seller, stock, bank eligibility, fees, and delivery may change by location or time; the partner's checkout is authoritative. Public product HTML is not a contracted marketplace feed. If a fetch cannot be verified, do not invent a price or label a cached snapshot live. The protected encryption master key is configured, but no Gemini or Groq provider API keys are present yet; show `Key required` until an administrator adds and verifies keys.

## Project structure
- `artifacts/dealnest/src/App.tsx`: customer app state, deal refresh, direct-link actions, and navigation.
- `artifacts/dealnest/src/data.ts` and `src/types.ts`: catalog, offer evidence fields, and exact source URLs.
- `artifacts/dealnest/src/components/`: product cards, outbound flow, admin dashboard, Operations & Health.
- `artifacts/dealnest/server.ts`: Express API routes, direct-item resolver, bounded safe retailer-page inspection and redirects, health checks and SPA serving.
- `artifacts/dealnest/server/liveDeals.ts`: bounded exact-product-page polling, verified price parser, five-minute cache and fallback state.
- `artifacts/dealnest/src/components/VerifiedDealFeed.tsx`: public Deals UI with exact links, checked prices, freshness and stale/unavailable states.
- `artifacts/dealnest/server/authService.ts`: salted admin PIN storage/validation, sessions, and rate limiting.
- `artifacts/dealnest/server/db.ts`: managed MySQL pool and bounded connection setup.
- `artifacts/dealnest/server/aiKeyStore.ts`: additive schema setup, AES-GCM key encryption/decryption, and safe key-metadata CRUD.
- `artifacts/dealnest/server/aiProviders.ts`: Gemini/Groq test and chat adapters with timeouts and sanitized errors.
- `artifacts/dealnest/src/components/AdminAiApiKeys.tsx`: admin-only form/list for provider keys; no plaintext key is returned after save.
- `artifacts/dealnest/src/components/AdminDashboard.tsx`: admin navigation and authorization wiring for the AI key panel.
- `artifacts/dealnest/server.ts`: admin key routes and ordered provider failover for the Concierge.
- `Dockerfile`: production install/build/server startup.
- `TODO.md`: requested product outcomes and complete acceptance clauses.

## Design
- **Design movement:** retain the uploaded expressive neo-brutalist commerce-dashboard style.
- **Core principles:** evidence before urgency; visible product actions; strong contrast and hierarchy; operational truth without hiding unavailable data.
- **Color philosophy:** preserve deep-indigo surfaces for focused comparison, electric lime for primary deal actions, cyan/emerald for healthy status, and amber for caveats and stale/fallback data.
- **Layout paradigm:** keep the existing discovery-first commerce dashboard and product-first deal cards; keep Operations & Health within the existing admin workspace.
- **Signature elements:** bold outlined cards with offset shadows; lime savings chips; compact source/freshness badges adjacent to offers.
- **Interaction philosophy:** exact mapped items open directly; unsupported links do not fabricate a destination; refresh and admin health actions explain what was actually checked.
- **Animation:** keep the existing brief card/modal transitions and subtle loading indicators; never let motion obscure stale or failed states.
- **Typography system:** preserve the current display/body pairing and condensed uppercase labels; render prices in the existing bold display style.
- **Brand essence:** DealNestAI compares source-backed listings for deal-seeking shoppers; **direct, skeptical, helpful**.
- **Brand voice:** concise, checkout-aware. Examples: “Open this exact item on Flipkart.” “Price check updated at 1:42 PM; confirm seller and payment offers at checkout.”
- **Wordmark & logo:** preserve the uploaded DealNest SVG and its existing stylized wordmark treatment.
- **Signature brand color:** DealNest electric lime `#ccff00`.

Direct source records and snapshots observed on 2026-10-10 (India retailer pages; examples for validation, not future price guarantees):
- Amazon iPhone 17 512 GB, Lavender base-option page: [exact item](https://www.amazon.in/iPhone-17-512-Promotion-Resistance/dp/B0FQF9TJ65), ₹109,999 / ₹124,900 MRP in the fetched result at 13:54 IST. Runtime checks reject the displayed Protect+/AppleCare bundled variant rather than treating its bundle price as the standalone price.
- Flipkart iPhone 17 512 GB, Mist Blue: [exact item](https://www.flipkart.com/apple-iphone-17-mist-blue-512-gb/p/itm4eaa82f5f27dd?pid=MOBHQV9YJQFBTGUK&marketplace=FLIPKART), observed at ₹109,999 at 13:54 IST.
- Amazon Sony WH-1000XM5: [exact item](https://www.amazon.in/Sony-WH-1000XM5-Wireless-Cancelling-Headphones/dp/B09XS7JWHH), observed at ₹19,489 at 13:54 IST.
- Flipkart Sony WH-1000XM5: [exact item](https://www.flipkart.com/sony-wh-1000xm5-wireless-industry-leading-active-noise-cancelling-headphones-mic-bluetooth-wired/p/itm9ee097bc0ae76), observed at ₹19,989 at 13:54 IST.
- Flipkart AirPods Pro (2nd generation, USB-C): [exact item](https://www.flipkart.com/apple-airpods-pro-2nd-generation-magsafe-case-usb-c-bluetooth-headset/p/itm60c8f5a308352), observed at ₹19,990 with a ₹23,900 MRP at 14:11 IST. No coupon, card, exchange, or add-on price is included in these base-price records.

The runtime checker refreshes mapped public item pages no more often than the five-minute cache window. Marketplace response time is measured separately from product-page price availability; both measurements are public-webpage observations, not official partner API SLAs.

The user's [Flipkart short link](https://dl.flipkart.com/s/zaQwLSuuuN) was browser-resolved on 2026-10-10 to the exact AirPods Pro 2 USB-C item page above. The item's structured product JSON-LD reports SKU `ACCGTCYRNADXCXNG`, INR 19,990. The local Preview now resolves the plain link, trailing-slash plus tracking-query form, and same slug under a Flipkart subdomain to this exact item and price. An unmapped link returns actionable guidance after a bounded redirect attempt; the final product identity is checked before any price is shown.


## Production release and access note
Version `b566e5872ccdc644ee0d4c4dabf10f710a803b1f` is published at https://dealnestai-4pojczz4.manus.space. Production `/_app/health` and `/manus-routes.json` returned HTTP 200. A signed-in administrator verified that application health and public configuration both report OK; retailer homepage reachability was Amazon 117 ms (HTTP 200), Flipkart 759 ms (HTTP 200), Zomato 482 ms (HTTP 200), and Zepto 27 ms (HTTP 202). These timings are public webpage reachability checks, not partner API SLAs. The production feed displayed three source-verified product-page prices (Flipkart iPhone 17 512 GB ₹1,09,999, Sony WH-1000XM5 ₹19,990, and AirPods Pro 2 USB-C ₹19,990); the Amazon iPhone page required browser verification and the Amazon Sony page returned HTTP 503, so neither was shown with an unverified price. Seed catalog prices were not used as live quotes.

The additional Flipkart short-link normalization and unknown-link guidance passed local Preview tests and is included in the published version above. The final production end-user Inspector flow remains unverified: `accessGateEnabled` is true and `requireCurrentDisclaimer` requires a user session that accepted the current disclaimer. An administrator session does not satisfy that user-session check. Although a prior admin inventory showed zero enrollment codes, the subsequent production login showed one active, unused `Standard Access Pass`; the visible audit trail contained only admin-login entries and no code-issuance event. We did not create, use, or revoke that code, and signed out. Preserve it and keep the gate enabled. The newly authorized `DNTEST` creation is paused for reconfirmation because the active-code state differs from the earlier check.

The user approved managed database enablement and the Gemini/Groq key manager. Managed MySQL is enabled at configuration revision 10; the protected `DEALNEST_AI_KEY_ENCRYPTION_KEY` is configured for All environments at secret revision 1 and runtime sync is applied. No API-key manager code or Gemini/Groq provider API keys are present yet, so neither provider can make live calls until an administrator adds valid keys. The user wants both provider adapters always included (Gemini then Groq sequential failover, not duplicate parallel calls) and no Gemini/Groq branding or provider-copyright text in the customer Concierge; preserve real source links and DealNest's own copyright. Both provider names may remain in the admin key manager. The live website remains on version `b566e5872ccdc644ee0d4c4dabf10f710a803b1f`.
