# VTEX Interview — Candidate Answer Scripts (Read-Aloud Ready)

Each answer below is written the way I'd actually say it out loud in the interview — first person, natural spoken rhythm, no bullet-point reading. Practice these by reading them aloud 2-3 times so they sound like your own words, not memorized lines. Swap in your real project/client details wherever I've written something generic.

---

### Q: "Can you walk me through VTEX's architecture — how does it all fit together?"

**Answer:**
"Sure. I usually think about VTEX's architecture in four broad layers. At the top, you've got the **Merchant Channels** layer — that's the storefront, mobile apps, POS systems, any marketplace where the merchant is actually selling. Below that is the **VTEX Core Services** layer — this is where Catalog, Pricing, Promotions, Checkout, Order Management, Master Data, and Logistics live. Then there's the **Back Office** layer — the merchant's own systems, like ERP, WMS, CRM — systems VTEX has to integrate *into*, not systems VTEX owns. And finally there's the **Third-Party** layer — payment gateways, tax engines, external marketplaces, and any custom microservices we build on top.

In my own project, for example, I worked mostly at the intersection of the Core Services layer and the Third-Party layer — we built a Node.js service that sat outside VTEX IO, listened for order events, and synced that data down to the client's ERP. So I've had hands-on exposure to how those layers actually talk to each other, not just the theory."

---

### Q: "Tell me about VTEX IO — what are the core building blocks?"

**Answer:**
"VTEX IO is VTEX's development platform, and everything in it is structured as an **app**. Every app is a versioned, git-like deployable unit, and it's defined by a `manifest.json` file, which is really the identity card of the app — it holds the name, vendor, version, and importantly, the **builders** it uses.

Builders are what tell VTEX what *kind* of code the app contains — so you'd have `react` for frontend components, `node` for backend services, `store` for store configuration, `graphql` for typed resolvers, and so on. On the frontend side, VTEX has something called the **Store Framework**, which lets you compose pages declaratively out of nested React components called blocks — you configure them with props rather than rewriting full pages. And then there's the alternative — going fully **headless**, where you use the same VTEX APIs but build a completely custom frontend, say in Next.js, instead of using the Store Framework's block system.

The trade-off I always explain to stakeholders is: Store Framework gets you to market faster because a lot of the UI composition is already there. Headless gives you a much higher ceiling on performance and design control, but you own a lot more of the engineering effort."

---

### Q: "Explain workspaces to me, and how do you handle promoting to production?"

**Answer:**
"A workspace in VTEX is basically an isolated, sandboxed copy of the entire account — you can think of it like a git branch, but for the whole store, not just code. Every account has one special workspace called `master`, and that's production — that's what real customers see.

My typical development flow is: I create or switch to a workspace using `vtex use`, then I run `vtex link`, which gives me hot-reload development — and importantly, that's only visible to me, it doesn't affect anyone else. Once I'm happy with the change, I don't just push it straight to master. What I do is either release and publish the app version and install that specific version into master, or — if it's a broader set of changes across the whole workspace — I use workspace promotion, so the entire tested workspace gets swapped into master atomically.

The key principle I always follow is: **you never hotfix master directly.** You validate everything in a workspace first, and only promote once you're confident it's stable. That protects production from half-tested changes."

---

### Q: "What's Master Data in VTEX, and when would you use it?"

**Answer:**
"Master Data is VTEX's native document database — it's schemaless, or close to it, and it's meant for storing custom data that doesn't fit into VTEX's standard commerce entities. So, things like extended customer profiles, custom checkout fields, loyalty points, store locations — anything specific to a client's business that VTEX doesn't model out of the box.

The way it works is you define a **Data Entity** — say a two-letter acronym like `LP` for loyalty points — and you attach a JSON **schema** to it that describes the fields, their types, which ones are required, and the schema version. Then you interact with it through the Master Data v2 REST API — standard CRUD, `GET`, `POST`, `PATCH`, `DELETE` against `/api/dataentities/{acronym}/documents`, and you can filter and sort using query parameters like `_where`, `_fields`, and `_sort`.

In practice, if someone asked me 'how would you store custom customer data VTEX doesn't natively support,' my answer is: create a new Data Entity with a schema, wrap it with a Node service or GraphQL resolver so the frontend doesn't talk to Master Data directly, and consume it from the React block or checkout flow. I'd only reach for an external database instead of Master Data if I needed something Master Data isn't built for — heavier relational queries, or data volumes beyond its rate and payload limits."

---

### Q: "What is broadcasting in VTEX?"

**Answer:**
"Broadcasting in VTEX refers to the event notification mechanism, mostly used around Order Management. Whenever an order changes state — placed, invoiced, cancelled, whatever — VTEX can broadcast that event out to external systems in near real time, instead of those systems having to constantly poll the OMS API.

There are two ways I've implemented this. One is an **OMS Hook** — you register a callback URL against specific order statuses, and VTEX fires an HTTP POST to that URL whenever an order hits that status. The other is through **VTEX IO's App Events** — internal events like `orders.OrderCreated` — which a Node.js service running inside VTEX IO can subscribe to directly, without needing an external webhook endpoint at all.

The real use case I've worked on is order sync to an ERP — when the order status changes, that broadcast triggers our integration service, which transforms the payload and pushes it downstream. The important design detail is that these broadcasts aren't guaranteed exactly-once — the same event can arrive twice if the receiving end is slow or fails and it retries — so I always build the handler to be idempotent, typically by checking the order ID and status against what's already been processed before acting again."

---

### Q: "How would you edit or update order details after an order is placed?"

**Answer:**
"VTEX's OMS REST API supports updating parts of an order after it's placed — for example, updating shipping information, adding tracking numbers once a shipment goes out, updating invoice data, or cancelling the order entirely. There are also fields like `marketingData`, `openTextField`, or custom data fields where you can attach additional information captured at checkout and read or write that back later for downstream systems like fulfillment or the ERP.

So a typical flow I've built is: capture a custom field at checkout — say a gift message or a delivery instruction — store it against the order, and then when the fulfillment team or an external WMS needs it, they pull it back out through the order API."

---

### Q: "Is it possible to link the master workspace — say, for order broadcast or order editing?"

**Answer:**
"That's a good question to be precise about. Order data itself, and OMS operations, are **account-level**, not workspace-scoped — orders live in the production account regardless of which workspace's app code happens to be processing them. What actually *is* workspace-scoped is which version of your integration app is running.

So the way I'd frame it: you don't 'link' order data to a workspace. What you do is develop and test your broadcast or order-editing integration logic in a dev workspace, validate it thoroughly there, and then promote that app — or promote the workspace — into master, so it becomes the version of the code that's actually handling live orders. The order data doesn't move; the *code that touches it* gets promoted."

---

### Q: "What's the difference between manifest.json and service.json?"

**Answer:**
"They serve two different purposes, and I think it's important not to mix them up. `manifest.json` sits at the root of every VTEX IO app and defines the app's *identity* — its name, vendor, version following semantic versioning, which builders it uses, its dependencies, and its policies — things like outbound access, which determine what external hosts the app is allowed to call.

`service.json`, on the other hand, only exists inside the `node` or `dotnet` folder, for apps that actually run backend services. It defines the *infrastructure configuration* for that service specifically — the routes it exposes, memory and CPU allocation, timeout settings, replica counts. Then those route IDs declared in `service.json` get implemented as actual middleware handlers in the `index.ts` file.

So the short way I'd put it: `manifest.json` is what the app *is*, `service.json` is how the backend service inside it actually *runs*."

---

### Q: "Walk me through how you'd create and deploy a new VTEX app."

**Answer:**
"I'd start with `vtex init`, or clone one of VTEX's boilerplate repos, which sets up the folder structure along with the `manifest.json`. I'd update the `name`, `vendor`, and `version` fields for my scenario, and declare whichever builders I need — `react` if it's a frontend block, `node` if it needs a backend service.

From there, I log in to the target account, switch to a development workspace with `vtex use`, and run `vtex link` — that gives me live hot-reload development, visible only to me, so I can iterate safely without touching anyone else's environment. Once the app is stable, I'd run `vtex release` to bump the semantic version, then `vtex publish` to make that version available in the registry for my vendor account. To actually get it live, I'd install that specific version into the target workspace — and if it's ready for production, promote that workspace into master, rather than linking directly onto master."

---

### Q: "What's schema in VTEX — what do you mean when you say 'schema'?"

**Answer:**
"It depends on the context, honestly, because VTEX uses the word 'schema' in a few different places, and I try to be precise about which one someone means. There's the **Master Data entity schema** — a JSON schema defining the fields, types, and required properties for a custom data entity. There's **catalog schema**, which covers product specifications and category structures. There's the `settingsSchema` field inside `manifest.json`, which VTEX uses to auto-generate an admin settings UI for your app based on a JSON Schema you define. And if we're talking about a `graphql` builder service, there's the actual **GraphQL schema** — the type definitions in the `.graphql` files.

So if someone asks me about schema without more context, I'd actually ask them to clarify which one they mean, because giving a precise answer matters more than guessing."

---

### Q: "Tell me about your experience with VTEX marketplace."

**Answer:**
"VTEX has a native marketplace model — the idea is multiple sellers can list products under a single storefront, and when an order comes in, it gets split per seller using something like a `sellerOrderId`, so each seller can handle their own fulfillment and logistics independently.

Where I've been hands-on with this is building **marketplace connector** integrations — syncing catalog data like SKUs, prices, and stock levels out to an external marketplace, pulling orders placed on that external marketplace back into VTEX's OMS, and then syncing status changes back out once those orders are fulfilled. The seller generally owns pricing, commission terms, and stock, while the marketplace account aggregates everything into one customer-facing storefront.

I'd also distinguish this from **Trade Policies**, or Sales Channels, which is a related but different concept — that's about applying different pricing, assortment, or payment rules per audience or channel, not necessarily involving multiple sellers."

---

### Q: "Give me a rapid overview — what's a Trade Policy, what's Intelligent Search, what's License Manager?"

**Answer:**
"Sure — quick hits on each:

A **Trade Policy**, also called a Sales Channel, is a rules set that controls pricing, payment methods, and shipping for a specific audience or channel — so you can run different pricing for, say, a wholesale audience versus retail, on the same catalog.

**Intelligent Search** is VTEX's native search and discovery engine — it handles indexing, facets, and ranking for the storefront's product search.

**License Manager** is VTEX's role and permission system — it controls who has access to what in the admin, and it's also what governs Data Entity permissions in Master Data, down to the field level if needed.

And **IOClients** are typed HTTP clients you use inside a Node service to call VTEX's own APIs or external ones — they handle a lot of the boilerplate around authentication and headers for you."

---

### Q: "Tell me about a project — walk me through what you built."

**Answer (template — fill in your real details before saying this):**
"On my last project, I was working with a [fashion / grocery / B2B — your real domain] client, and I was primarily responsible for [frontend on the Store Framework / backend Node services / both]. One thing I built that I'm proud of was [specific: e.g., a custom product recommendation block, a checkout customization, an ERP order-sync integration].

The architecture decision I made there was [e.g., choosing to build it as a headless block calling a custom GraphQL resolver rather than modifying the core catalog block directly], and I chose that because [reason — performance, maintainability, avoided forking a core VTEX app].

The main challenge I ran into was [e.g., workspace conflicts when two developers were linking simultaneously, or a Master Data schema migration that broke existing documents], and I solved it by [your actual solution].

The impact was [concrete: faster page load, fewer support tickets, successful go-live on time, etc.]."

**Practice note:** Say this one out loud with your *actual* project details filled in until it takes under 90 seconds and doesn't sound read — this is the single most important answer in the whole interview.

---

## How to use this for practice
1. Read each answer aloud slowly once, just to hear the correct rhythm and pronunciation.
2. Read it a second time without looking, filling gaps in your own words.
3. Time yourself — most of these should land between 45–90 seconds spoken. Longer than that and you'll lose the interviewer's attention.
4. For pronunciation of specific terms (VTEX, GraphQL, PostgreSQL, etc.), refer back to the pronunciation table in the Tech Lead JD prep doc — practice those words in isolation before running through full answers.
