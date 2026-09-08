# Backend Interview Prep — Node.js + VTEX IO (Basic to Medium)

Format: direct answer first, then a bit more depth. Read the direct answer as your opening line in the interview, then continue naturally into the explanation.

---

## PART 1 — Node.js Fundamentals (Basic)

### Q: What is Node.js?
**Answer:** Node.js is a JavaScript runtime built on Chrome's V8 engine that lets you run JavaScript outside the browser, on a server. It's single-threaded but handles concurrency through an event-driven, non-blocking I/O model, which makes it efficient for I/O-heavy applications like APIs and web servers.

### Q: What is the event loop?
**Answer:** The event loop is what allows Node.js to perform non-blocking I/O despite being single-threaded. When an async operation like a file read or a network call is issued, Node.js hands it off to the system, continues executing other code, and when that operation completes, its callback is pushed into a queue. The event loop continuously checks this queue and executes callbacks once the call stack is empty.

Deeper detail if pushed: the event loop has phases — timers, pending callbacks, idle/prepare, poll, check, and close callbacks — and microtasks (like resolved Promises) run between each phase, before the next phase begins.

### Q: Difference between synchronous and asynchronous code in Node.js?
**Answer:** Synchronous code executes line by line, blocking further execution until the current operation finishes. Asynchronous code lets Node.js continue executing other code while waiting for an operation — like a database call — to complete, and handles the result later via a callback, Promise, or async/await.

### Q: What's the difference between callbacks, Promises, and async/await?
**Answer:** They're three evolving ways to handle asynchronous code. Callbacks are functions passed as arguments, executed once an operation completes — but nesting many of them creates "callback hell." Promises represent a future value with `.then()` and `.catch()`, which flattens the nesting and gives cleaner error handling. Async/await is syntactic sugar over Promises — it lets you write asynchronous code that reads like synchronous code, using `try/catch` for errors, which is what I use day to day since it's the most readable.

### Q: What is `require` vs `import`?
**Answer:** `require` is Node's original CommonJS module system — synchronous, and used with `module.exports`. `import`/`export` is the ES Module syntax, which is asynchronous under the hood and is now natively supported in modern Node versions, or via Babel/TypeScript transpilation in older setups. On VTEX IO Node services, I've typically used `import`/`export` since the builder compiles TypeScript.

### Q: What is npm, and what's the difference between dependencies and devDependencies?
**Answer:** npm is Node's package manager, used to install and manage third-party libraries. `dependencies` are packages needed at runtime, in production. `devDependencies` are only needed during development — like testing libraries, linters, or type definitions — and aren't included when the app is built for production.

### Q: What is middleware in the context of a Node backend (like Express)?
**Answer:** Middleware is a function that sits in the request-response cycle, with access to the request, response, and a `next` function to pass control to the next middleware. It's used for things like authentication checks, logging, parsing request bodies, or error handling, and it lets you compose cross-cutting concerns without repeating that logic in every route handler.

### Q: How do you handle errors in async Node.js code?
**Answer:** With async/await, I wrap the call in a `try/catch` block, catch the error, log it with context, and either return a proper error response or re-throw it depending on where in the flow I am. For unhandled cases, I make sure there's a global error handler at the top level so nothing crashes the process silently — an unhandled promise rejection can crash a Node process if not caught.

### Q: What are environment variables and why do we use them?
**Answer:** Environment variables store configuration — API keys, database URLs, feature flags — outside the codebase, so the same code can run in different environments (dev, staging, production) without hardcoding secrets or config values into the source. In Node, they're accessed via `process.env`.

---

## PART 2 — Node.js Fundamentals (Medium)

### Q: What is the difference between `process.nextTick()`, microtasks, and `setTimeout()`?
**Answer:** `process.nextTick()` callbacks run immediately after the current operation completes, before the event loop continues — they have the highest priority. Microtasks, like resolved Promise callbacks, run right after `nextTick` callbacks, also before the event loop moves to its next phase. `setTimeout()`, even with a delay of 0, is scheduled in the timers phase of the event loop, so it runs after all pending microtasks and nextTick callbacks. In short: nextTick, then microtasks, then timers.

### Q: What are Streams in Node.js, and why would you use them?
**Answer:** Streams let you process data piece by piece instead of loading the entire data set into memory at once. There are four types — Readable, Writable, Duplex, and Transform. I'd use them for things like reading a large file, processing a big CSV import, or piping data from one source to another, because it keeps memory usage low and lets processing start before the whole payload has arrived.

### Q: What is an EventEmitter?
**Answer:** EventEmitter is a core Node.js class that implements the observer pattern — you can emit named events and have multiple listener functions react to them. It's the foundation a lot of Node's own APIs are built on, like Streams and HTTP servers, and I've used it when building internal pub-sub style logic within a service, decoupling the event trigger from the handling logic.

### Q: How would you prevent or debug a memory leak in a Node.js service?
**Answer:** Common causes are global variables holding onto references, uncleaned event listeners, or caches that grow unbounded. To debug it, I'd take heap snapshots — using something like Node's built-in `--inspect` flag with Chrome DevTools, or a tool like `clinic.js` — and compare snapshots over time to see which objects are accumulating without being garbage collected. Prevention-wise, I make sure to remove event listeners when they're no longer needed, and set reasonable size limits or TTLs on any in-memory caches.

### Q: What's the difference between `Cluster` and `Worker Threads` in Node.js?
**Answer:** Both are ways to use multiple CPU cores, since Node is single-threaded by default. `Cluster` forks multiple full Node processes that share the same server port, each with its own memory and event loop — good for scaling web servers horizontally on one machine. `Worker Threads` run actual threads within a single process, sharing memory more efficiently via `SharedArrayBuffer`, and are better suited for CPU-intensive tasks like heavy computation, since you don't want to block the main event loop with that kind of work.

### Q: How do you structure error handling and retries when calling an external API from Node?
**Answer:** I wrap the call in try/catch, distinguish between different failure types — timeout, 4xx client error, 5xx server error — and handle them differently. For transient failures like timeouts or 5xx errors, I implement a retry with exponential backoff and a maximum attempt limit, rather than retrying immediately or infinitely. For 4xx errors, I generally don't retry, since retrying a bad request won't fix it — I'd log it and surface a clear error instead.

### Q: What testing approach do you use for Node.js backend code?
**Answer:** I write unit tests for individual functions and business logic using something like Jest, mocking external dependencies like database calls or API clients so tests run fast and deterministically. For integration-level confidence, I test the actual route/handler behavior, checking the response shape and status codes. I try to keep business logic separate from the framework/routing layer specifically so it's easier to unit test in isolation.

---

## PART 3 — VTEX IO Backend Specific (Basic)

### Q: How do you create a backend service on VTEX IO?
**Answer:** You use the `node` builder — declared in `manifest.json` — which lets your app run server-side Node.js code inside VTEX IO. Inside the `node` folder, `service.json` defines the routes and infrastructure config, like memory allocation and timeout, and `index.ts` implements the actual route handlers as middleware functions.

### Q: What is `service.json` and what goes in it?
**Answer:** It's the infrastructure configuration file for a Node or dotnet service inside a VTEX IO app. It defines the routes the service exposes by ID, along with memory, CPU, timeout, and replica settings. Those route IDs are then implemented as handler functions in the actual code file.

### Q: What are IOClients in VTEX IO?
**Answer:** IOClients are typed HTTP client classes provided by the `@vtex/api` package that let a Node service call VTEX's own Core Commerce APIs — like Catalog, Master Data, or OMS — or external third-party APIs, with built-in handling for authentication headers, retries, and caching. Instead of writing raw fetch or axios calls, you extend a base client class and define typed methods, which keeps API calls consistent and reusable across the app.

### Q: How do you call an external API (outside VTEX) from a VTEX IO Node service?
**Answer:** You need to declare an `outbound-access` policy in `manifest.json`, specifying the allowed host and path — VTEX IO blocks outbound calls by default for security, so this is required before your service can reach that host at all. Then in code, you'd use an IOClient extending `ExternalClient` to make the actual call.

### Q: How does logging work in a VTEX IO Node service?
**Answer:** You use `ctx.vtex.logger`, which is the context-aware logger VTEX IO provides — it automatically tags logs with account, workspace, and request context, which makes it much easier to trace an issue back to a specific request or environment compared to a plain `console.log`.

### Q: What's the difference between `manifest.json` and `service.json`?
**Answer:** `manifest.json` is at the app root and defines the app's identity — name, vendor, version, builders it uses, dependencies, and policies like outbound access. `service.json` is specific to the `node`/`dotnet` folder and defines the backend service's own routes and infrastructure settings. Manifest is what the app is, service.json is how the backend service inside it runs.

---

## PART 4 — VTEX IO Backend Specific (Medium)

### Q: How do you handle an event triggered by a VTEX platform action, like an order status change, in a Node service?
**Answer:** VTEX IO supports App Events — internal events like `orders.OrderCreated` — which a Node service can subscribe to by declaring an event handler in `service.json` and implementing the corresponding function. When the event fires, VTEX IO invokes that handler with the event payload, and I can then run custom logic — for example, transforming and pushing the order data out to an external system. The key design point I always keep in mind is that these events aren't guaranteed exactly-once, so the handler needs to be idempotent — checking whether it's already processed this exact event before acting again.

### Q: How would you design a Node service in VTEX IO that needs to sync data to an external ERP?
**Answer:** I'd subscribe to the relevant order or catalog event, and on trigger, transform the VTEX payload into the shape the ERP expects. I'd wrap the outbound call in retry logic with backoff for transient failures, and log failures clearly so they're traceable. If the external system can be down for extended periods, I'd also think about whether we need a dead-letter or manual reprocessing path, rather than retrying forever silently. And critically, I'd make sure the operation is idempotent on the ERP side too, if possible — using the order ID as an idempotency key — so a duplicate delivery doesn't create duplicate records downstream.

### Q: What are the tradeoffs of using VTEX's `graphql` builder versus a `node` builder with REST routes?
**Answer:** The `graphql` builder lets you define a typed schema and resolvers, and the storefront can request exactly the fields it needs in a single request — which is efficient for frontend-driven data fetching with varying requirements per page. A `node` builder with REST routes is simpler to reason about for fixed-shape integrations, like a webhook receiver or a straightforward proxy to an external API, where the frontend doesn't need flexible querying. On my project, I've used node/REST primarily for backend-to-backend integration work, since GraphQL's real advantage shows up more on the frontend-facing side.

### Q: How do you handle configuration or secrets for a VTEX IO Node service, like an API key for an external system?
**Answer:** VTEX IO doesn't let you hardcode secrets in code that gets deployed, since the app bundle isn't meant to hold sensitive credentials. The typical approach is to use a Configuration app pattern — a separate app with a `configuration/schema.json` that defines the settings shape, which the service app then reads at runtime via a `read-workspace-apps` policy. This keeps secrets out of the versioned app code and lets different accounts configure their own credentials.

### Q: How would you debug a production issue in a VTEX IO Node service?
**Answer:** First, I'd check the logs via `ctx.vtex.logger` output, filtering by account, workspace, and time window to narrow down the failing requests. I'd try to reproduce the issue in a development workspace using `vtex link`, rather than debugging live against master, so I'm not risking further production impact. Once I isolate the root cause, I'd fix and test it thoroughly in the dev workspace, and only then promote the fix through to master, following the same release discipline as any other change.

### Q: What's your understanding of how VTEX IO handles scaling for Node services — is it something you manage yourself?
**Answer:** VTEX IO is a serverless PaaS for the `node` builder — I don't manage servers or provision infrastructure directly. Scaling and replica management are largely abstracted by the platform, though `service.json` does let you configure things like minimum/maximum replicas and memory allocation, so there's some control over the resource envelope, but not infrastructure-level management like you'd have with a self-hosted Node service on AWS with Docker and Jenkins.

---

## Quick tips before the interview

- If asked a question you genuinely don't know cold, it's fine to say: *"I haven't hit that specific scenario, but based on how [related concept] works, I'd expect it to behave like..."* — reasoning from principles is respected more than a wrong guess stated confidently.
- For any "how would you design X" question, always mention: error handling, idempotency, and logging/observability — these three come up as follow-ups constantly at Tech Lead level even if not asked directly.
- Practice saying "IOClients," "manifest dot jason," "service dot jason" out loud a couple of times — see the pronunciation guide from the earlier Tech Lead JD prep doc if you need a refresher.
