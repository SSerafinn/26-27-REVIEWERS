# Midterm Reviewer — Systems Integration and Architecture

**Unit V — Integration Techniques · Unit VI — API Design & Management**
School of Information Technology and Engineering (SITE), SPUP

> **How to use this reviewer:** This is a summary of the main ideas, not a replacement for the lectures. For each topic, make sure you can do three things: **define** the term, **explain why** it exists, and **choose** it correctly in a new situation. The self-check questions at the end test exactly that.

---

## Part 1 — Unit V: Integration Techniques

### 1.1 Synchronous vs. asynchronous integration

Every integration makes one timing decision: does the caller wait for an answer or not?

| Aspect | Synchronous | Asynchronous |
|---|---|---|
| **Caller** | Sends a request and waits for the response | Hands the message to a queue or broker and continues |
| **Coupling** | Tighter — both sides must be up at the same time | Looser — the broker holds the message until it is processed |
| **When something is slow** | The caller is slowed down too | The queue absorbs the delay |
| **Good for** | Answers the caller needs right now | Long work, notifications, sending one event to many systems |
| **Main cost** | Fragile when services are slow or down | Harder to trace and debug |
| **Typical form** | A REST API call | A message queue or event stream |

> **Remember:** Ask, *does the caller need the result to continue?* If yes, synchronous. If not, asynchronous is usually more resilient.

### 1.2 Topologies vs. patterns

Topologies (from Unit II: point-to-point, hub-and-spoke, ESB, API-led) describe the **shape** of the connections between systems. Integration patterns describe **what happens along** those connections. A useful image: topology is the road network; patterns are the traffic rules.

### 1.3 Enterprise Integration Patterns

Named by Gregor Hohpe and Bobby Woolf in the book *Enterprise Integration Patterns*. The names are a shared vocabulary that lets architects describe a design quickly.

| Pattern | What it does | Think of it when… |
|---|---|---|
| **Message Channel** | The pipe that carries messages between systems | Two systems need a path to exchange messages |
| **Content-Based Router** | Reads each message and sends it to the right destination | Different messages must go to different places |
| **Message Translator** | Converts a message from one format to another | Two systems disagree on format or structure |
| **Aggregator** | Combines related messages into one | Many pieces must become one whole |
| **Splitter** | Breaks one message into several | One whole must become many pieces |
| **Dead Letter Channel** | Holds messages that cannot be processed | Bad messages must be set aside for review |

> **Remember:** Splitter and Aggregator are opposites. A Router changes **where** a message goes; a Translator changes **what it looks like**.

### 1.4 Data integration techniques

Some integration moves data between stores rather than messages between applications.

| Technique | How it works | Best suited for |
|---|---|---|
| **ETL** | Extract, transform, then load the finished data into the target | Strict formats; smaller volumes |
| **ELT** | Extract, load the raw data, then transform inside the warehouse | Powerful cloud warehouses; keeping raw data to reprocess later |
| **Change Data Capture (CDC)** | Reads the database's change log and streams every insert, update, and delete as it happens | Keeping other systems in step in near real time |
| **Replication** | Copies data to another store and keeps the copy current | Reporting copies that protect the live system; standby copies for failover |

> **Remember:** ETL and ELT usually run in **batches** (nightly, hourly). CDC is **continuous**. The difference between ETL and ELT is **where and when** the transformation happens.

### 1.5 Designing for scale and resilience

Production systems face slow networks, restarts, and partial failures. Four principles help integrations survive them.

**Loose coupling.** Systems depend on each other only through stable interfaces, not shared databases or internal details. Then one system can change, restart, or scale without breaking the others. Everything below builds on this.

**Idempotency.** An operation is idempotent if doing it many times has the same effect as doing it once. It matters because a lost reply leaves the caller unsure whether its request worked, so its only safe move is to retry. The usual tool is an **idempotency key**: the client sends a unique ID with the request; if the server has already processed that ID, it returns the original result instead of repeating the work.

**Retries with exponential backoff.** Retry failed calls, but wait longer each time (for example 1, 2, 4, 8 seconds) and add a little randomness, called **jitter**, so that many clients do not retry at the same instant. Retrying immediately and endlessly turns a small problem into a flood.

**Circuit breakers.** When a service keeps failing, stop calling it for a while so callers fail fast instead of waiting.

| State | Behavior | Moves to… |
|---|---|---|
| **Closed** | Normal: calls pass through; failures are counted | Open, when failures pass a threshold |
| **Open** | Calls are blocked and fail immediately | Half-open, after a timeout |
| **Half-open** | One trial call is allowed through | Closed if the trial succeeds; Open if it fails |

> **Remember:** Retries need idempotency (or work gets duplicated), backoff keeps retries from becoming a flood, and circuit breakers stop one failing service from dragging down everything that calls it — the cascading failure from Unit II.

---

## Part 2 — Unit VI: API Design & Management

### 2.1 APIs and REST

An **API** is a contract that lets one system ask another for data or actions without knowing how it works inside. It is loose coupling made concrete: as long as the contract holds, both sides can change their internals.

**REST** (Representational State Transfer) is an architectural style described by Roy Fielding in his 2000 doctoral dissertation. It dominates because it reuses ordinary HTTP, which every language, browser, proxy, and firewall already understands.

| Constraint | Meaning |
|---|---|
| **Client–server** | The client and the server can evolve separately |
| **Stateless** | Every request carries everything the server needs; the server keeps no memory of earlier calls — this lets any server handle any request, which is what makes scaling out easy |
| **Cacheable** | Responses say whether they can be reused |
| **Uniform interface** | Every resource is handled the same way, with the same small set of methods |

### 2.2 HTTP methods

A **safe** method changes nothing on the server. An **idempotent** method has the same effect whether sent once or many times.

| Method | Purpose | Safe? | Idempotent? |
|---|---|---|---|
| **GET** | Read a resource | Yes | Yes |
| **POST** | Create a new resource | No | No |
| **PUT** | Replace a whole resource | No | Yes |
| **PATCH** | Change part of a resource | No | Not guaranteed |
| **DELETE** | Remove a resource | No | Yes |

> **Remember:** POST is the method that is **neither safe nor idempotent** — which is why APIs that take payments add idempotency keys to POST requests.

### 2.3 Designing resources and URLs

- Name resources with **nouns**, not verbs — the HTTP method is the verb.
- Use **plural** names for collections, such as `/v1/books`.
- Address one item by its ID, such as `/v1/books/7`.
- Nest a resource under its parent only when it truly belongs to it, and keep nesting to **two levels or fewer**, such as `/v1/authors/3/books`.
- If a URL contains a word like *get*, *create*, or *delete*, the design is fighting HTTP instead of using it.

### 2.4 Status codes

The status code is the first thing a client checks. **2xx** means success, **4xx** means the client made a mistake, and **5xx** means the server failed.

| Code | Name | Use it when… |
|---|---|---|
| **200** | OK | A read or update succeeded and returns data |
| **201** | Created | A POST created a resource (include its Location) |
| **204** | No Content | It succeeded, but there is nothing to return, as with many deletes |
| **400** | Bad Request | The request is malformed or missing required fields |
| **401** | Unauthorized | The caller is not authenticated |
| **403** | Forbidden | The caller is authenticated but not allowed |
| **404** | Not Found | The resource does not exist |
| **409** | Conflict | The request clashes with the current state of the data |
| **422** | Unprocessable Content | The request is well formed but breaks a validation rule |
| **429** | Too Many Requests | The caller exceeded its rate limit |
| **500** | Internal Server Error | Something failed inside the server |
| **503** | Service Unavailable | The server is overloaded or down for maintenance |

> **Common confusions:**
> - **401 vs. 403:** 401 is "I don't know who you are"; 403 is "I know who you are, and the answer is still no."
> - **400 vs. 422 vs. 409:** 400 — I can't read this; 422 — I can read it, but it breaks a rule; 409 — it's valid, but it clashes with what already exists.
> - **Never** return 200 with an error message inside: every client, monitor, and retry policy will think it succeeded.

### 2.5 API design practices

**Versioning.** Lets you make breaking changes without forcing every client to update at once. Adding something optional usually needs no new version; renaming, removing, or changing the type of something clients use does.

| Approach | Example | Trade-off |
|---|---|---|
| **URL path** | `/v1/books` | Most visible and most common |
| **Request header** | `API-Version: 2026-10-01` | Clean URLs, but easy to miss |
| **Query parameter** | `/books?version=2` | Simple, but mixes versioning with filtering |

**Pagination.** Large collections are returned in pages.

| Style | How it works | Trade-off |
|---|---|---|
| **Offset / limit** | Skip a number of items, then return a page | Simple and allows jumping to any page; can skip or repeat items when data changes, and slows down on large data |
| **Cursor** | Continue after a bookmark the server returned | Stable and fast on large, changing data; cannot jump straight to a far page |

**Error handling.** Pair the right status code with one consistent error body for the whole API: a machine-readable **code** for programs, and a human-readable **message** for developers.

**Filtering and sorting.** Use query parameters on a collection, such as `?category=fiction&sort=-year`, instead of inventing a new endpoint for every question.

### 2.6 Webhooks

With **polling**, the client keeps asking "anything new?" — wasting calls and still learning late. With a **webhook**, the client registers a URL once, and the provider calls it the moment an event happens. A reliable webhook receiver does three things:

1. **Verifies the sender**, usually by checking an HMAC signature made with a shared secret.
2. **Responds fast and processes later**, so the provider does not time out and resend.
3. **Is idempotent**, recording each event ID and ignoring repeats, because providers retry.

### 2.7 Beyond REST: GraphQL and gRPC

| Aspect | REST | GraphQL | gRPC |
|---|---|---|---|
| **Origin** | Fielding, 2000 | Facebook; open-sourced 2015 | Google; open-sourced 2015 |
| **Shape** | Many resource URLs | One endpoint; the client writes a query | Procedures in a `.proto` contract |
| **Data format** | Usually JSON | JSON | Protocol Buffers (binary) |
| **Transport** | HTTP | HTTP, usually POST | HTTP/2, with streaming |
| **Strength** | Simple, cacheable, universal | Client gets exactly the fields it needs in one trip | Very fast and compact |
| **Watch out for** | Over- or under-fetching data | Harder caching; expensive queries need limits | Poor fit for browsers and public APIs |
| **Best fit** | Public APIs, standard CRUD | Many clients needing different data | Internal service-to-service calls |

### 2.8 API gateways and management

An **API gateway** sits in front of many services and handles shared concerns in one place:

- **Authentication** — checks keys or tokens once, for every service.
- **Rate limiting** — caps how many requests a client may make in a period.
- **Routing** — sends each request to the right service.
- **Logging and metrics** — one view of traffic, errors, and latency.
- **Circuit breaking** — the Unit V pattern, applied at the edge.

**Token bucket.** Each client has a bucket of tokens that refills at a steady rate. Each request spends one token; an empty bucket means the request is refused with **429 Too Many Requests**. A full bucket allows short bursts, while the refill rate enforces the long-run average. A good 429 response includes a **Retry-After** header, and a good client waits that long and backs off with jitter.

**Rate limiting vs. throttling.** Often used as synonyms. When distinguished, rate limiting **rejects** extra requests, while throttling **slows or queues** them.

**Developer portal.** A website where outside developers read the documentation, register apps, get API keys, and see their usage.

### 2.9 Documenting with OpenAPI

The **OpenAPI Specification** — originally the **Swagger** specification, donated in 2015 to the OpenAPI Initiative under the Linux Foundation — is the standard machine-readable description of a REST API, written in YAML or JSON. It lists every endpoint, parameter, request and response shape, and status code. Tools turn it into interactive documentation (Swagger UI), generate client code, and test that an API still matches its contract. Writing the OpenAPI file first and building to match it is called **contract-first** design.

---

## Part 3 — How Units V and VI Connect

Many Unit VI ideas are Unit V principles expressed in HTTP. Seeing the link helps you answer "why" questions.

| Unit V principle | Where it appears in Unit VI |
|---|---|
| **Loose coupling** | The API as a stable contract; versioning so clients are not broken |
| **Idempotency** | Safe and idempotent HTTP methods; idempotency keys on POST; idempotent webhook receivers |
| **Retries with backoff and jitter** | Honoring Retry-After after a 429 response |
| **Circuit breakers** | Circuit breaking applied at the API gateway |
| **Asynchronous integration** | Webhooks: the provider notifies you instead of you polling |
| **Message Translator** | A gateway or adapter converting between formats and versions |

---

## Key Terms

| Term | Meaning |
|---|---|
| **Synchronous / asynchronous** | Whether the caller waits for the response or hands the message off and continues |
| **Message broker / queue** | Middleware that holds messages until the receiver processes them |
| **ETL / ELT** | Transform before loading, versus loading first and transforming in the warehouse |
| **Change data capture** | Streaming every insert, update, and delete from a database's change log |
| **Loose coupling** | Depending on other systems only through stable interfaces |
| **Idempotent** | Repeating an operation has the same effect as doing it once |
| **Idempotency key** | A unique request ID that lets a server recognize and ignore a retry |
| **Exponential backoff / jitter** | Waiting longer between each retry / adding randomness to those waits |
| **Circuit breaker** | Stops calls to a failing service, then tests it with one trial call |
| **Resource** | A noun in an API, such as a book or member, with its own URL |
| **Stateless** | The server keeps no memory of earlier requests |
| **Safe method** | A method that changes nothing on the server |
| **Webhook** | An HTTP call the provider makes to your URL when an event happens |
| **HMAC signature** | A code computed with a shared secret that proves who sent a message |
| **API gateway** | A single entry point that applies shared policies and routes requests |
| **Token bucket** | A rate-limiting method where requests spend tokens that refill steadily |
| **OpenAPI** | The standard machine-readable description of a REST API |

---

## Self-Check Questions

Try to answer each question in your own words before checking the answer guide that follows.

1. In one sentence each, when should an integration be synchronous, and when asynchronous?
2. Name the Enterprise Integration Pattern you would use to (a) change a message's format, (b) send messages to different destinations, and (c) set aside messages that keep failing.
3. What is the difference between ETL and ELT, and why has ELT become popular?
4. Why is CDC described as "near real time" while ETL is not?
5. Explain why retries and idempotency must go together.
6. Describe what a circuit breaker does in each of its three states.
7. Why does REST's stateless constraint make it easier to scale an API across many servers?
8. Which HTTP methods are idempotent? Which one is not, and why does it matter?
9. Give a URL that breaks REST naming rules and rewrite it correctly.
10. Explain the difference between 401 and 403, and between 409 and 422.
11. Which changes to an API require a new version, and which usually do not?
12. What are the three habits of a reliable webhook receiver, and which Unit V principle does each one apply?
13. When would you choose GraphQL over REST? When would you choose gRPC?
14. Explain how a token bucket allows short bursts but still limits the average rate.
15. What does an OpenAPI file describe, and name two things tools can do with it.

## Self-Check Answer Guide

1. Synchronous when the caller needs the answer to continue; asynchronous when the work is long, can happen later, or must reach several systems.
2. (a) Message Translator, (b) Content-Based Router, (c) Dead Letter Channel.
3. ETL transforms before loading; ELT loads raw data first and transforms inside the warehouse. Modern cloud warehouses can do the transformation themselves, and keeping raw data allows reprocessing later.
4. CDC streams each change as it happens from the change log; ETL usually runs in scheduled batches, so data is only as fresh as the last run.
5. A lost reply forces a retry, and without idempotency the retry can repeat the work — such as charging twice.
6. Closed: calls pass and failures are counted. Open: calls are blocked and fail fast. Half-open: one trial call decides whether to close again or reopen.
7. Because no server holds session memory, any server can handle any request, so more servers can simply be added behind a load balancer.
8. GET, PUT, and DELETE are idempotent; POST is not. Retrying a POST can create duplicates, so critical POSTs need idempotency keys.
9. Example: `GET /getBookById?id=7` becomes `GET /v1/books/7`. Use nouns and plurals, and let the method be the verb.
10. 401: not authenticated; 403: authenticated but not allowed. 409: valid but conflicts with existing data; 422: well formed but breaks a validation rule.
11. Renaming, removing, or changing the type of something clients use needs a new version. Adding optional fields or new endpoints usually does not.
12. Verify the signature (security), reply fast and process later (asynchronous work), and ignore repeated event IDs (idempotency).
13. GraphQL when many clients need different data and you want fewer, exact requests. gRPC for fast, compact calls between internal services, especially with streaming.
14. A full bucket holds many tokens, so a burst can spend them at once; after that, requests are limited by the steady refill rate, and an empty bucket returns 429.
15. Every endpoint, parameter, request and response shape, and status code. Tools can generate interactive documentation, generate client code, and test that the API matches its contract.

---

> **Exam tips:** Read every option before choosing. In scenario questions, find the **key detail** first — what does the caller need, what failed, what changed? Then match it to the concept. If two options look right, pick the one that solves the **root cause**, not just the symptom.
