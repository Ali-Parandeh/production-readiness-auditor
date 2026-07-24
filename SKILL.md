---
name: production-readiness-auditor
description: Run a structured production-readiness audit on a FastAPI plus LLM/Generative AI codebase, grounded in the book Building Generative AI Services with FastAPI (O'Reilly, 2025). Built to run inside a repository from a coding agent such as Claude Code. It walks the codebase itself and asks only what the code cannot reveal. Use whenever the user asks to audit or review a FastAPI or GenAI repo, asks "is my codebase production-ready", wants a FastAPI or LLM code review, mentions a "production checklist" or "GenAI deployment check", or points the agent at a FastAPI/LLM service. Also trigger for questions about onion or modular architecture, FastAPI routing and API design, LLM integration and streaming, async concurrency, LLM security and optimisation, behavioural testing of LLM apps, or Docker and container security in a Python context, even without the word "audit". Works through seven areas one at a time, grading each Pass/Partial/Fail with book-grounded feedback, then a final overview and matched O'Reilly reading.
compatibility: Optional O'Reilly MCP (endpoint https://api.oreilly.com/api/content-discovery/v1/mcp/ , docs https://learning.oreilly.com/apidocs/mcp/content/) for book-grounded feedback and recommendations. Degrades to references/oreilly-fallback.md when not connected.
---

# Production Readiness Auditor

Audit a FastAPI + Generative AI codebase against seven areas drawn from *Building Generative AI Services with FastAPI* (Ali Parandeh, O'Reilly, 2025). Each area maps to specific chapters.

This is built to run **inside the repository** from a coding agent. The agent inspects the code itself; it does not ask the developer to paste or describe what it can read. It works through the seven areas **one at a time**, grading each and reporting before moving on, and pauses to ask the developer **only** when the code leaves a genuine ambiguity that would change the grade. After all seven, it gives one final overview.

## How to run the audit

1. **Map the repo.** Read the directory tree and locate the key files (entry point, routers, services, models, settings, `Dockerfile`, `tests/`, dependency manifests). Confirm it is a FastAPI + LLM/GenAI service. Tell the developer this runs as seven short rounds and starts with Area 1.
2. **Loop through the seven areas in order** using the per-area loop below, reading the relevant files for each area as you reach it.
3. **Final overview** once all seven are graded.
4. **Write the report file** to the repo root (see "Writing the report file").

You have file tools (glob, grep, read, write). Use them. Read the files an area names in its **Inspect** line rather than reading the whole repo at once.

## The per-area loop (repeat for each of the seven areas)

1. **Announce** the area: "Area N of 7 — [name] (Ch X)", with one line on what it covers.
2. **Ground yourself.** If the area names a reference file, read it first so feedback uses the book's framework and vocabulary.
3. **Inspect the code.** Walk the files in the area's **Inspect** line: glob for the relevant paths, grep for the signals, read what matters. Form a provisional finding from what you actually see, citing file and line.
4. **Ask only what the code cannot tell you.** Compare what you found against the grounding, and ask the developer only about genuine gaps: intent, scale, deployment, or things that live outside the repo (see each area's **Confirm** line). If the code answers everything, ask nothing. Never ask about something you can read. Cap at three questions, and only when they would change the grade. If you ask, **stop and wait**.
5. **Report.** Once resolved, give the **per-area report**:
   - **Finding:** what is or is not there, with file and line.
   - **Grade:** Pass / Partial / Fail.
   - **Fix:** the specific change, grounded in the book's guidance with the chapter reference.
   - **Recommended reading (Partial/Fail only):** the specific book chapter for this area, plus, if an O'Reilly MCP search tool is connected, one or two resources it returns for this area's gap. Skip entirely for a Pass. Keep it to a line or two.
6. Move to the next area.

Keep each per-area report short. Per-area reading is one or two lines for a failed area; the consolidated, prioritised list is saved for the final overview, not repeated each round.

## Grounding the feedback

Feedback should reflect the book, in this order of reliability:

1. **Reference files** (`references/chNN-*.md`). When an area names one, it holds the book's actual frameworks and vocabulary for that area. Primary grounding, always available.
2. **O'Reilly MCP content** (supplement, when available). Tools vary by account. If the answers tool `ask_oreilly_experts` is present, query it scoped to this book and area to enrich or confirm the reference, and use `get_oreilly_citation` to pull the exact source for anything you quote. Summarise into your own words; keep any direct quote short and attribute it with the chapter and a link.
3. **Built-in checks** in each area. Self-sufficient on their own.

**Never fabricate book content.** State only what the reference files or the MCP actually contain. If neither covers a point, use the built-in checks and say the feedback is general rather than from the book.

The MCP keyword search tool (such as `search_oreilly_content`) is used for the **recommendations** step in the final overview regardless of whether a content tool exists. If no O'Reilly MCP is connected, recommendations come from `references/oreilly-fallback.md`.

---

## Area 1 — Modular project architecture (Onion design) · Ch 2

**Read `references/ch02-architecture.md` before grading.**

Judge two things: does the **project structure** (flat / nested / modular) fit the service's complexity, and is the **onion design** applied with dependencies pointing inward.

Check for: a structure appropriate to complexity (a complex GenAI backend should not still be flat); dependencies pointing inward, so domain models and business logic do not import the web framework or data-access detail; the onion layers present and separated — **API routers, controllers, services, repositories, schemas/models**; **services distinguished from providers** (providers wrap external systems such as model APIs, email, payments); data access behind **repositories** rather than inline in controllers; cross-cutting components where needed — **middleware, dependencies, pipes, mappers, exception filters, guards**; configuration via `pydantic-settings`/environment, not hardcoded.

Common fails: a complex service still in a flat layout; a single fat `main.py`; LLM or external calls made directly in route handlers; business logic in controllers; no repository layer; secrets or model names hardcoded; circular imports.

Inspect: the directory tree; `main.py` (does it hold everything, or delegate?); imports in domain/service modules (do they import `fastapi`?); presence of a `modules/`, `services/`, `repositories/`, `routers/` split; `settings.py` / `pydantic-settings` usage.

Confirm with the developer (only if unclear): whether the current structure is deliberate for the project's expected size and lifespan.

Grade: Pass / Partial / Fail. Borderline cases: can the layout be justified to another developer?

---

## Area 2 — FastAPI routing and API design · Ch 2, type-safety Ch 4

**Use `references/ch02-architecture.md`** for the router/controller/exception-filter detail.

Check for: Pydantic models for request bodies **and** responses, with `response_model` set; `APIRouter` used to group routes and apply shared logic, grouped by resource; controllers that **inject** data and logic via `Depends` rather than constructing it inline; a versioned surface (for example `/v1`); correct status codes and a consistent error envelope via **exception filters/handlers**, not ad hoc dicts per route; validation handled by Pydantic (Ch 4 type-safety), not scattered manual checks; meaningful OpenAPI docs (summaries, examples); pagination or limits on list endpoints.

Common fails: routes returning raw dicts with no `response_model`; no central exception handling; everything hung off the app object with no `APIRouter`; controllers building dependencies inline instead of injecting them; no versioning; unvalidated free-form input passed straight to the model.

Inspect: route decorators for `response_model`; `APIRouter(...)` usage and prefixes; `Depends(` in handler signatures; `@app.exception_handler` / handler registration; path prefixes for versioning; the request/response Pydantic models.

Confirm with the developer (only if unclear): whether an unversioned or sparsely documented API is a deliberate choice for an internal-only service.

Grade: Pass / Partial / Fail.

---

## Area 3 — LLM integration and real-time streaming · Ch 3, Ch 5, Ch 6

**Read `references/ch06-streaming.md` before grading the streaming half**, and `references/ch05-async.md` for the serving-strategy rationale. Model loading mechanics and monitoring (Ch 3) are not yet grounded; treat those specifics as general until a Ch 3 reference exists.

How models are loaded and served (Ch 3, Ch 5), and how output is streamed in real time (Ch 6).

Check for (integration and serving): models integrated through the **application lifespan** rather than loaded at import time; a serving strategy matched to the model, hosted-provider calls using an async client, but a **self-hosted large model externalised** (vLLM, Ray Serve, Triton, or a separate server) rather than loaded in-process where memory-bound inference would block the server (see `ch05-async.md`); monitoring of model calls via middleware.

Check for (streaming, Ch 6): tokens **streamed as generated** (provider `stream=True` returning a generator, passed through FastAPI), not buffered until the full completion is ready; the mechanism (SSE vs WebSocket) chosen with a rationale fit to the workflow, one-way generation suiting SSE and interactive/bidirectional suiting WebSocket; for WebSocket, a **connection manager** tracking active connections, graceful close with `WebSocketDisconnect` handled, and errors sent as **WebSocket messages and CLOSE-frame codes, not HTTP status codes**; a **single streaming entry point** that switches behaviour via headers/body/query params, rather than many preconfigured streaming endpoints the client must hop between.

Common fails: model loaded at module import (slow cold start, duplicated across workers); no streaming, so the client waits for the whole response; SSE vs WebSocket chosen with no rationale; a WebSocket route with no connection manager or no graceful disconnect handling; WebSocket errors returned as HTTP exceptions; a proliferation of streaming endpoints for a single conversation; no monitoring on model calls.

Inspect: `lifespan=` / `@asynccontextmanager` / legacy `@app.on_event("startup")`; where the model or client is instantiated (module top level vs lifespan); provider `stream=True`; `StreamingResponse`, SSE (`EventSourceResponse` / `text/event-stream`), or `@app.websocket`; a connection-manager class holding `active_connections`; `WebSocketDisconnect` handling; how many distinct streaming endpoints exist.

Confirm with the developer (only if unclear): which model(s) and whether hosted or local, if not evident; whether the interaction is one-way or bidirectional, which justifies SSE vs WebSocket; response sizes and latency expectations.

Grade: Pass / Partial / Fail.

---

## Area 4 — Async concurrency for GenAI · Ch 5

**Read `references/ch05-async.md` before grading.** The rule is not "be async everywhere", it is **never block the event loop**. A plain `def` handler with sync code is fine (FastAPI thread-pools it); a blocking or sync call inside an `async def` handler is the cardinal fail, freezing every other request.

Check for: `async def` handlers that contain **only** non-blocking, awaited calls; an async client (`httpx.AsyncClient` or the provider's async SDK) used inside async handlers, never `requests` or a sync SDK; unavoidable sync or CPU-bound work pushed off the loop via `run_in_executor`/`anyio.to_thread`; for hosted-provider calls (inference is I/O-bound for you), the **async client** plus **exponential backoff/throttling** for rate limits (for example `stamina`); for self-hosted models (inference is memory-bound), **externalised serving** (vLLM, Ray Serve, Triton, or a separate server) rather than loading the model in-process; long-running inference handled without blocking, light jobs via `BackgroundTasks` and heavy inference via an external server or queue (Ray Serve, BentoML, vLLM, or Celery + Redis + RabbitMQ); timeouts on external calls; bounded concurrency and closed connections to avoid leaks.

Common fails: a blocking or sync call inside an `async def` handler; `requests` or a sync SDK in an async path; forgetting to `await`; a large self-hosted model loaded in-process so inference blocks the server; heavy CPU-bound inference dumped into a background task expecting parallelism (it runs on the same event loop); no backoff on external rate limits; unbounded concurrency leaking memory.

Not a fail (do not flag): a synchronous `def` handler with a sync client (FastAPI thread-pools it); a simple or early-stage service that is synchronous; background tasks used for genuinely light work.

Inspect: handler definitions (`async def` vs `def`) and whether async handlers hold only non-blocking awaited calls; client type (`httpx.AsyncClient`/async SDK vs `requests`/sync SDK) and whether it matches the handler; `time.sleep` or sync I/O inside coroutines; `run_in_executor`/`anyio.to_thread`; `BackgroundTasks` vs an external queue/server; backoff or `stamina` on external calls; concurrency caps and connection cleanup; whether models are self-hosted in-process or externalised (vLLM/Ray/Triton).

Confirm with the developer (only if unclear): whether the service calls a hosted provider (I/O-bound, async is the lever) or self-hosts models (memory-bound, needs externalised serving or multiprocessing); expected concurrent load and latency targets.

Grade: Pass / Partial / Fail.

---

## Area 5 — LLM security and optimisation · Ch 8, Ch 9, Ch 10

**Read `references/ch08-auth.md`, `references/ch09-security.md`, and `references/ch10-optimisation.md` before grading.** One combined assessment covering auth, security/guardrails, and optimisation.

Check authentication and authorisation (Ch 8): an auth layer on protected routes, with the **method matched to the deployment context** (Basic only for prototype/internal; Token/JWT for SPA, mobile, REST; OAuth for enterprise SSO or external identity data; key-based for small/internal); role-based access control through the dependency graph where actions need restricting by role; the authn-vs-authz distinction respected (verifying identity is not the same as verifying permission).

Check security (Ch 9): rate limiting on public and model-backed endpoints; **input guardrails** against the categories the book names — topical, direct prompt injection/jailbreak, indirect prompt injection (sanitising external/encoded content), moderation (profanity, competitor, explicit, PII, self-harm), and attribute validation (length, size, range, format); **output guardrails** — hallucination/fact-check, moderation, and syntax checks (validating JSON schemas and function/tool selections); guardrails run **in parallel to inference** (asyncio) where they add latency, cancelling generation and returning a canned response on trigger; secrets/keys read from the environment, never from string literals.

Check optimisation (Ch 10): **batch processing** where throughput matters (structured-output lists or a provider batch API via JSONL); **caching matched to the input pattern** (keyword for exact repeats, semantic for similar-meaning queries, context/prompt for large repeated context), watching the semantic-cache trap of returning one answer where outputs should vary; **quantization** only where models are self-hosted; and a **structured system prompt** (the book's Role-Context-Task template) rather than ad hoc prompts scattered through the code.

Common fails: keys in code or baked into images; no auth on production or external routes; Basic auth on an internet-facing service (plain-text credentials); no input guardrails against prompt injection on a user-facing endpoint; guardrails run sequentially so they double the latency; thresholds left untuned; repetitive high-traffic workloads with no caching.

Inspect: auth dependencies/middleware and which method (Basic / JWT / OAuth / key); RBAC role checks via `Depends`; rate-limit middleware (for example `slowapi`); input and output guardrail calls (topical/injection/moderation/attribute; hallucination/syntax/JSON-schema validation), and whether they run via `asyncio.wait` parallel to inference; how keys are read (env vs literal); caching (`fastapi-cache`, `gptcache`, Redis, semantic or context cache) and batch API or JSONL batch use; prompt templates vs inline strings.

Confirm with the developer (only if unclear): the deployment context (internal/prototype vs production/external), which sets the acceptable auth method and guardrail strictness; where secrets are injected at runtime; whether rate limiting or guardrails are handled upstream by a gateway; expected traffic, which determines the value of caching and batching, and whether parallel guardrails risk provider rate-limiting.

Grade: Pass / Partial / Fail.

---

## Area 6 — Tests: mocked LLM tests and behavioural testing · Ch 11

**Read `references/ch11-testing.md` before grading.** Scale the verdict to stakes: thin tests on a prototype are not a fail, but thin tests on a minimum sellable product, a service handling sensitive data, or one processing payments are.

Check for: a test suite proportional to the stakes, with static checks (`mypy`) and a sensible spread of unit, integration, and E2E tests structured as given-when-then (arrange-act-assert); **LLM and external calls mocked** (monkeypatch, `respx`, fixtures) and substituted via dependency injection, so the suite is deterministic and does not hit a paid API in CI; tests that check **behaviour, not implementation detail** (they should survive a behaviour-preserving refactor).

For the non-determinism of model output, check for **behavioural / property-based testing** rather than exact-match assertions, covering the three categories the book uses:
- **Minimum Functionality Tests (MFTs):** correct behaviour on simple, well-defined inputs and failure modes (grammar, known facts, zero toxicity, rejecting inappropriate input, readable output).
- **Invariance Tests (ITs):** output stays consistent under irrelevant input changes (case, whitespace, typos, synonyms, number formats, reordering).
- **Directional Expectation Tests (DETs):** output moves in the right direction as input changes (sentiment addressed, more complex prompt yields a longer/more complex response).

Also look for **auto-evaluation** tests (a discriminator or LLM-as-judge scoring hallucination, toxicity, relevancy against a threshold), a **regression/monitoring** plan for model drift (especially with hosted providers), and, for internet-facing services, **adversarial tests** that also exercise the auth guards.

Common fails: deterministic exact-match assertions on model output (flaky in CI); tests that call the live model every run; no mocking or DI; HTTP-status assertions only, with nothing checking output properties; tests bound to implementation detail that break on refactor; no regression/monitoring for a service on a hosted model.

Inspect: `tests/`, `conftest.py`, pytest config in `pyproject.toml`/`pytest.ini`; `mypy` config; mocking of LLM/external calls (`respx`, `monkeypatch`, fixtures, DI overrides); whether output assertions check properties (length, readability, toxicity, JSON validity) or exact strings; presence of MFT/IT/DET-style or auto-evaluation tests; CI config; any drift monitoring.

Confirm with the developer (only if unclear): the stakes (prototype vs MSP vs sensitive/payment handling), which sets the bar; whether the suite runs in CI; whether any live-model tests are deliberate.

Grade: Pass / Partial / Fail.

---

## Area 7 — Docker and container security · Ch 9, Ch 12

**Read `references/ch12-deployment.md` before grading.** Separate security checks (which can fail the area) from efficiency ones (which are weaknesses to flag, not fails).

Security: runs as a **non-root** user (root is a real risk, a compromised container then has host root); **no secrets baked into layers** (`ENV`, `ARG`, copied `.env`), injected at runtime instead; a **`.dockerignore`** excluding `.env`, `.git`, `__pycache__`, `.venv`, caches, so `COPY . .` does not leak secrets or bloat the image; pinned/locked dependencies.

Efficiency: a **minimal base image** (`slim` for build time, `alpine` for size), not `latest` or a full distro in production; a **multi-stage build** that leaves build tooling out of the production image (the dev/prod/`--target` pattern); **layer ordering** that copies `requirements.txt` and installs **before** `COPY . .`; combined `RUN` instructions; explicit `EXPOSE` and `HEALTHCHECK`.

Model and storage (where relevant): large models **externalised** (volumes in dev, external storage or persistent volumes in prod, loaded at startup) rather than baked into the image, with health-check timing that tolerates startup downloads; if self-hosting on CPU, ONNX + quantization to shrink the image (Ch 10); runtime data and logs persisted to volumes rather than the **ephemeral** container layer.

Common fails: running as root; `FROM python:latest`; `COPY . .` pulling secrets in; no `.dockerignore`; unpinned dependencies; a single bloated stage; a multi-gigabyte image from baked-in models or a GPU runtime that CPU inference did not need; data written to the ephemeral layer with no volume.

Inspect: `Dockerfile` (base image tag slim/alpine vs latest/full; stages and `--target`; `USER`; layer order, requirements before `COPY . .`; combined `RUN`; secrets in `ENV`/`ARG`/`COPY`; `EXPOSE`; `HEALTHCHECK`); `.dockerignore` contents; whether large models are `COPY`ed in or loaded at startup; ONNX/quantization if self-hosting; volume or persistent-volume use; lockfiles; any `docker-compose`/k8s manifests.

Confirm with the developer (only if unclear): the deployment target (VM / cloud function / managed service / container) and how secrets are injected at runtime, which usually lives outside the repo; whether the service self-hosts models or calls a hosted API, which decides whether the model/quantization checks apply.

Grade: Pass / Partial / Fail.

---

## Final overview (after all seven areas)

Pull the seven grades together.

1. **Summary table:**

| Area | Chapter | Grade | One-line reason |
| --- | --- | --- | --- |
| 1 Modular architecture (Onion) | 2 | | |
| 2 FastAPI routing and API design | 2, 4 | | |
| 3 LLM integration and streaming | 3, 6 | | |
| 4 Async concurrency | 5 | | |
| 5 Security and optimisation | 8, 9, 10 | | |
| 6 Tests and behavioural testing | 11 | | |
| 7 Docker and container security | 9, 12 | | |

2. **Top three urgent fixes** across all areas, ordered by risk to production (secrets exposure and event-loop blocking outrank cosmetic issues). One sentence each, each pointing at the specific thing in the code.
3. **Overall read:** one short paragraph on how close this is to production and what stands between it and ready.
4. **Recommendations (consolidated).** Pull the per-area reading already surfaced during the audit into **one prioritised list**, ordered to match the top fixes above, rather than repeating each area verbatim. Always include *Building Generative AI Services with FastAPI* (O'Reilly, 2025) as the primary reference, with the specific chapter for each failed area. If an O'Reilly MCP search tool is connected, this is the place to round out resources across the gaps (terms specific to each failed area, for example "server-sent events streaming Python", "asyncio GIL", "semantic caching LLM", "prompt injection guardrails", "behavioural testing NLP CheckList", "Docker non-root container security"); otherwise draw from `references/oreilly-fallback.md`. Only Partial/Fail areas, matched to the actual gaps, not a generic list.

---

## Writing the report file

After the final overview has been rendered in chat, write the whole audit to a Markdown file at the **root of the repository**, so the developer keeps a durable record.

- **Filename:** `PRODUCTION_READINESS_AUDIT-{DATETIME}.md`, where `{DATETIME}` is the completion time as `YYYY-MM-DD-HHMM` in local time, with no colons (filesystem-safe). Example: `PRODUCTION_READINESS_AUDIT-2026-06-29-1430.md`. Do not overwrite earlier audits; the timestamp keeps each run distinct.
- **Header inside the file:** a title and the same timestamp, for example:
  - `# Production Readiness Audit`
  - `Generated: 2026-06-29 14:30 (local)`
  - the repository name or path
  - a one-line source note: based on *Building Generative AI Services with FastAPI* (O'Reilly, 2025), via the production-readiness-auditor skill.

**Contents: exactly what was rendered in the chat, and nothing else.**

- All seven **per-area reports** as shown, each with its Finding (with `file:line` references preserved verbatim), Grade, Fix, and Recommended reading.
- Any **developer answers to the Confirm questions** that affected a grade, recorded in the relevant area as a short line, for example "Developer confirmed: hosted provider, ~50 concurrent users." Capture the substance that changed the assessment, not the full back-and-forth.
- The **final overview**: the summary table, the top three urgent fixes, the overall read, and the consolidated recommendations.

**Do not include:** tool output, command results, grep or file-content dumps, reference-file contents, your own reasoning or planning, or any context that was not shown to the developer in the chat. The file mirrors the audit as the developer saw it, not the work behind it.

Preserve every `file:line` reference exactly as it appeared in the chat. After writing, tell the developer the file path.
