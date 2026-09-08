# VTEX Technical + Managerial Round — Prep Guide

## 0. About the "Logixel" search
I couldn't find any public Reddit/Glassdoor/LinkedIn threads with Logixel-specific VTEX interview questions — it's a smaller partner and doesn't have crowdsourced reports yet. What's below is built from VTEX's own official IO documentation plus the pattern of questions reported for VTEX Software Engineer roles on Glassdoor (mostly: your project narrative + e-commerce scenario questions + light DSA). Since Logixel is a **VTEX implementation partner**, their tech round will almost certainly test the same platform fundamentals below, not company trivia — so this should transfer directly.

---

## 1. How to structure "tell me about your projects" (this is the core of the round)

The hiring manager told you plainly: they want your **real work history**, told as a technical story, not just buzzwords. Prepare this in STAR format for 2–3 projects:

- **Client/domain** (fashion, grocery, B2B, marketplace, etc. — don't name confidential clients if restricted, describe the business type)
- **Your role**: were you frontend-only (React/VTEX IO Store Framework), backend-only (Node service), or full-stack across both?
- **What you built**: specific apps — e.g., a custom shelf/product block, a checkout customization, a custom Master Data entity + admin UI, an order broadcast integration with an ERP, a marketplace connector
- **Architecture decisions**: why you chose a certain approach (e.g., store framework block vs. a headless storefront calling VTEX APIs)
- **Challenges + how you solved them**: workspace conflicts, IO cache issues, GraphQL resolver performance, Master Data schema migrations, order status sync failures
- **Impact**: performance improvement, bug reduction, faster releases, successful go-live

Have **one story ready for**: a production bug you debugged, a time you disagreed with an architecture decision, and a project that didn't go well and what you learned. These three come up in almost every VTEX loop per Glassdoor reviews.

---

## 2. VTEX Architecture (the "architecture" topic they flagged)

Be ready to explain, at a whiteboard level:

### 2.1 Overall reference architecture (4 layers)
- **Merchant Channels** — storefront, mobile apps, POS, marketplaces selling the merchant's catalog
- **VTEX Core Services** — Catalog, Pricing, Promotions, Checkout, OMS (Order Management), Master Data, Logistics
- **Back Office** — ERP, WMS, TMS, CRM — systems VTEX integrates *into*
- **Third-Party** — payment gateways, tax engines, marketplaces, custom microservices

Know how to draw this and place a project you worked on inside it (e.g., "we built a Node service in the Third-Party layer that synced order status back to SAP").

### 2.2 VTEX IO (the dev platform) core concepts
- **Apps**: everything on VTEX IO is an app — a versioned, git-like deployable unit with `manifest.json`
- **Builders**: `react`, `node`, `store`, `styles`, `graphql`, `messages`, `docs`, `admin` — each declares what kind of code the app contains
- **Runtime**: the layer that resolves which app/block renders for a given route, per account+workspace
- **Store Framework**: declarative UI composition via `store.blocks.json`/theme apps — page templates built from nested "blocks" (React components) configured with props, no full page rewrites needed
- **Headless / Frontier**: same VTEX backend APIs (Catalog, Checkout, OMS) but a fully custom frontend (Next.js, etc.) instead of the Store Framework — know the trade-off: faster to launch with Store Framework, more control/performance ceiling with headless

### 2.3 Workspaces (they specifically flagged this)
- A **workspace** is an isolated, sandboxed copy of the account's apps/config — like a git branch for the whole store
- Every account has a `master` workspace = **production**
- Dev flow: `vtex use <workspace>` → creates/switches to a workspace → `vtex link` (dev mode, hot reload, only visible to you) → `vtex publish`/`vtex deploy` app version → **promote workspace to master** to go live
- **"Linking to master workspace"**: you generally don't link directly onto `master` in production practice — `vtex link` on master is disabled/discouraged; the safe pattern is: link on a dev workspace → test on `workspace--account.myvtex.com` → once stable, either (a) release/publish the app version and install it into master, or (b) for whole-workspace promotion, use `vtex workspace promote` to atomically swap a fully-tested workspace into master. Be ready to say this clearly: **you don't hotfix master directly; you promote a validated workspace into it.**
- Workspaces can be `production`-type (billed, used for real traffic like a staging domain) or regular dev workspaces (free, ephemeral)

### 2.4 CDN, caching, and multi-tenancy
- VTEX IO is multi-tenant; each account/workspace resolves at the edge via CDN
- Segment/cache headers, `no-cache` vs `x-vtex-*` headers, cache invalidation on link/publish — mention if you've debugged stale-cache issues

---

## 3. Node.js + React on VTEX (your day-to-day stack)

### React / Store Framework side
- Building custom **blocks** (functional React components) registered in `store.blocks.json` / `interfaces.json`
- Using `useRuntime()`, `useProduct()`, `useCssHandles()` from `vtex.render-runtime` / `vtex.css-handles`
- Content Provider apps (Site Editor–configurable blocks) vs pure code blocks
- GraphQL data fetching in the storefront via `react-apollo` / VTEX's GraphQL IO layer
- Styles builder (`styles.json`, CSS handles/tachyons overriding) — how you theme without forking core apps

### Node.js / Services side
- Writing a **service** (`node` builder) that exposes HTTP routes, GraphQL resolvers, or event handlers running server-side inside VTEX IO
- `service.json` (see section 6) defines routes; `index.ts`/`index.js` implements route handlers/middlewares
- Calling **VTEX Core Commerce APIs** (Catalog, Pricing, OMS, Master Data) from a Node service using `IOClients` — outbound calls need `policies` declared in `manifest.json` (host allow-listing)
- Event handlers: reacting to internal VTEX events (e.g., `orders.OrderCreated` via broadcast/appkey) to trigger custom Node logic
- Be ready to discuss: async/await error handling, rate limiting/backoff when calling VTEX APIs, logging via `ctx.vtex.logger`

---

## 4. "Data basecamp" → almost certainly **Master Data (MD)**

This is one of VTEX's most-asked platform topics. Be very solid here:

- **Master Data** = VTEX's schemaless/NoSQL-ish document database for custom data (customer profiles, custom entities like "Store Locations", "Loyalty Points", custom checkout fields)
- **Data Entities**: you define an entity (e.g., `CL` for clients, or a custom `LP` for loyalty points) with a **schema** (JSON) describing fields, types, required fields, and version
- **Master Data v2 API**: REST endpoints for CRUD — `GET/POST/PATCH/DELETE /api/dataentities/{acronym}/documents`
- Search: `_fields`, `_where`, `_sort`, pagination via `_page`/`_pageSize` or `_more`
- **Triggers/CallCenter operator/Scroll**: bulk operations, exporting large datasets
- Common real interview question: *"How would you store custom customer data not natively supported by VTEX?"* → Answer: create a new Data Entity + schema in Master Data, expose via a custom Node service/GraphQL resolver, and consume from the React block or checkout UI.
- Data Entity **permissions**: role-based access control per entity/field via License Manager
- Difference between Master Data and using an external database from your own Node service (when you'd choose one vs the other — MD is native/no infra to manage but has payload/rate limits; external DB gives full control but you own the ops)

---

## 5. Order broadcast, order editing, and OMS

This was explicitly flagged — spend real prep time here.

### Order Broadcast
- VTEX OMS can **broadcast order status changes** (created, invoiced, cancelled, etc.) via a **hook/webhook config** so an external system (ERP, WMS) is notified in near real time
- Configured typically via `orderHooks` / OMS Hook config (URL + filter on order status), or by subscribing to VTEX's **App Events** (`orders.OrderCreated`, `orders.OrderStatusChanged`) inside a Node app using `ctx.clients.events` / IO event handlers
- Retry/backoff behavior when the receiving endpoint is down — be ready to say how you'd design idempotent handling (the same order status ping may arrive more than once)

### Editing order details
- OMS REST API lets you **PATCH/update** parts of an order — e.g., change shipping data, add tracking info, update invoice data, cancel an order, add custom order data via `marketingData`/`openTextField`/`customData`
- Key endpoints to know by name (even loosely): `PUT /api/oms/pvt/orders/{orderId}`, invoice endpoint, cancellation endpoint, order status change endpoint
- **Custom order fields**: adding checkout custom fields captured at purchase time, then reading/writing them back via the order API for downstream fulfillment systems

### Linking master workspace / order sync question
If they literally ask "is it possible to link the master workspace [for order broadcast/edits]?" — the honest technical answer: order data and OMS operations are **account-level**, not workspace-scoped like app code — orders live in the production account regardless of which workspace's app code is handling them. What *is* workspace-scoped is which version of your integration app is running. So your integration app (the one doing the broadcast/edit logic) gets developed/tested in a dev workspace, then **promoted to master** so it's the one processing live orders — you don't "link" order data itself to a workspace, you promote the *code* that touches orders into master.

---

## 6. `service.json`, app creation, and "projection creation"

### App creation flow
1. `vtex init` or clone a boilerplate → sets up folder with `manifest.json`
2. `manifest.json` = app identity: `name`, `vendor`, `version` (semver), `builders` (declares app "type": react/node/store/etc.), `dependencies`, `policies` (outbound access, permissions), `settingsSchema`
3. `vtex login <account>` → `vtex use <workspace>` → `vtex link` (dev/live-reload) → iterate
4. `vtex release <major|minor|patch>` → bumps version + tags
5. `vtex publish` → makes the app version available in the VTEX App Store registry for the vendor account
6. Install the app into a workspace/account: `vtex install <vendor>.<appname>@<version>`
7. Promote workspace to master (or install directly into master) to go live

### `service.json` — specifically
- Lives inside the `/node` (or `/dotnet`) folder of an app that uses the `node`/`dotnet` builder
- Defines the **infrastructure config for the service**: the routes it exposes, memory/CPU allocation, timeout, min/max replicas
- Routes declared here (by route ID) are then implemented/matched to middleware handlers in `index.ts`
- Distinction to be crystal clear on in the interview: **`manifest.json`** = app-level identity & builders/policies (what the app *is*); **`service.json`** = the Node/dotnet service's own routing & infra config (how the service *runs*)

### "Projection creation" — likely means one of two things, be ready for either:
1. **Search projections in VTEX Intelligent Search / Catalog** — when you index a product, VTEX projects catalog data into a search-optimized index (facets, specifications) — relevant if they mean search/catalog indexing
2. **GraphQL projections** — selecting/shaping a subset of fields from a resolver's response (standard GraphQL field selection) — relevant if they mean the `graphql` builder services
If unsure which they mean live in the interview, it's fine to ask them to clarify — shows precision, not weakness.

### Configuration apps + schema.json (a specific, sometimes-asked flow)
- **Service apps** provide functionality; **Configuration apps** configure them
- A configuration app defines `configuration/schema.json` — a JSON Schema describing what settings shape it accepts
- The service app then reads that config at runtime via a `read-workspace-apps` policy
- This pattern is how many VTEX billing/settings/feature-toggle style apps work — good to mention if you've built a "Settings" admin page for a custom app

---

## 7. Schema (broader sense)

Different "schemas" that could come up — clarify which one they mean, or briefly cover all three:

1. **Master Data entity schema** — JSON schema defining fields/types/required for a Data Entity (section 4)
2. **Catalog schema / specifications** — product specification groups, SKU fields, category trees
3. **`manifest.json`'s `settingsSchema`** — JSON Schema used to render an admin settings UI for your app automatically
4. **GraphQL schema** — type definitions (`.graphql` files) for a `graphql` builder service

---

## 8. Marketplace

- VTEX supports a **native marketplace model**: a "seller hub" concept where multiple sellers list products under one storefront, orders get split per seller (`sellerOrderId`), each seller has its own fulfillment/logistics
- **Marketplace connector apps**: integrations that sync catalog/inventory/orders between VTEX and external marketplaces (Amazon, Mercado Livre, etc.) — if you've built one, know: catalog push (SKU/price/stock sync), order pull (fetching orders placed on the external marketplace into VTEX OMS), and status sync back out
- **Seller** vs **Marketplace (buyer org)** account relationship — pricing, commission, and stock are typically seller-owned; the marketplace account aggregates
- Distinguish this from **VTEX Sales Channels** (trade policies) — a related but different concept (different price/assortment/payment rules per channel, not necessarily multi-seller)

---

## 9. Rapid-fire concepts worth a one-line answer each

| Term | One-liner to have ready |
|---|---|
| **Trade Policy (Sales Channel)** | Rules set controlling price table, payment, shipping per audience/channel |
| **Checkout customization** | Custom checkout UI via checkout API/custom JS, or headless checkout via Checkout API |
| **IOClients** | Typed HTTP clients for calling VTEX/external APIs from a Node service |
| **App Store / VTEX Apps** | Public registry of installable apps (yours can be private-to-vendor or published) |
| **License Manager** | VTEX's role/permission system controlling admin & API access |
| **Intelligent Search** | VTEX's native search/discovery engine (facets, ranking, indexing) |
| **Headless CMS (VTEX)** | Content management for non-catalog content, API-driven |
| **CI/CD on VTEX IO** | Git-based app versioning + `vtex release`/`publish`, workspace promotion as your "deploy" |

---

## 10. Questions to ask the hiring manager (shows seniority)

- "What does the current architecture look like — mostly Store Framework, or has the team moved toward a headless setup?"
- "How is order broadcast / ERP sync currently handled — event-driven or polling?"
- "How does the team manage workspace promotion and rollback if something breaks in master?"

---

**Bottom line prep priority for tonight**: (1) polish your 2–3 project stories with specifics, (2) be able to draw the 4-layer architecture + workspace flow from memory, (3) nail Master Data CRUD + schema, (4) know the manifest.json vs service.json distinction cold, (5) have a clear answer ready for the order-broadcast/master-workspace question since they called it out by name.
