# Chapter 5 — Concurrency, async, and model serving

Grounding for **Area 4** (async concurrency) and the serving-strategy half of **Area 3**. Source: *Building Generative AI Services with FastAPI*, Ch 5.

## The mental model

Two kinds of blocking operation:
- **I/O-bound:** waiting on network, database, file, or user. Async and multithreading help here because the CPU can do other work while waiting.
- **Compute-bound:** CPU/GPU-intensive work like AI inference. Async and multithreading do **not** help; the processor is busy computing.

Concurrency (interleaving tasks on one core via async or threads) suits I/O-bound work. Parallelism (multiple cores via multiprocessing) suits compute-bound work.

The single most important rule: **never block the event loop.** Correctness is not "use async everywhere", it is "do not run blocking work on the loop".

## async def vs def in FastAPI (key calibration)

FastAPI runs on ASGI (Starlette) and serves requests via both an event loop (async) and a thread pool (sync handlers). It handles both:
- An **`async def` handler** must only do non-blocking, awaited work. FastAPI trusts you here. A blocking or synchronous call inside an `async def` handler **blocks the event loop**, freezing every other request until it finishes, whether the blocking is I/O- or compute-bound. This is the cardinal fail.
- A plain **`def` handler** with synchronous code is **fine**: FastAPI hands it to the thread pool so it does not block the loop. It finishes a little slower than an async equivalent (threads must acquire the GIL), but it is correct. The FastAPI docs explicitly say not to over-worry about `async def` vs `def`.

So: do **not** flag a sync `def` handler with a sync client as a fail. **Do** flag a sync/blocking call inside an `async def` handler. The book's three-way example: `async` handler calling a sync client = blocks the loop (worst); `def` handler with a sync client = thread-pooled (fine); `async` handler with `await async_client...` = non-blocking (best).

If async work has unavoidable synchronous or CPU-bound parts, push them off the loop with a thread or process pool executor (`run_in_executor` / `anyio.to_thread`).

## Async correctness pitfalls

- `await` only works inside `async def`; every async call must be awaited.
- Mixing sync into async negates the benefit: using `requests` instead of an async client, forgetting `async`/`await`, or calling a sync-only library (for example a sync DB driver) inside an async handler. Verify a library's async support before relying on it; if it is sync-only, switch to an async equivalent.
- Unclosed connections/buffers and unbounded concurrency cause memory leaks; bound the number of concurrent operations.
- Race conditions and deadlocks violate thread-safety. The book's advice: start synchronous, then migrate to async once the logic is understood.

## External provider APIs vs self-hosted models

- **Hosted provider (OpenAI, Anthropic, Mistral, etc.):** inference is **I/O-bound for you**; you offload the compute to the provider over the network. Async is the right lever; use the provider's **async client** so FastAPI processes requests concurrently. Throttle concurrent calls to stay within rate limits using **exponential backoff** (the `stamina` package helps), or request higher limits.
- **Self-hosted large model:** inference is **memory-bound** (loading billions of parameters into GPU high-bandwidth memory dominates), not something async fixes. Loading the model in the same process as the FastAPI server blocks the server during inference. Multiprocessing workers do **not** share memory, so each worker reloads the model and burns hardware. The answer is to **externalise model serving**, not parallelism alone: host it behind another server or a specialised inference server (vLLM, Ray Serve, NVIDIA Triton) that handles batching, tensor parallelism, quantization, KV/paged attention, GPU memory management, and streaming.

## Long-running inference (queuing)

Some models (e.g. SDXL) take minutes; simultaneous users create a backlog. FastAPI **background tasks** let you respond immediately and process in the background, then save results to disk/DB, offer polling, or push updates over a live connection.

Critical warning: **background tasks run in the same event loop and give concurrency, not parallelism.** Heavy CPU-bound inference in a background task still blocks the loop until done; an async background task that fails to await its blocking I/O also blocks. Non-async background tasks run in FastAPI's internal thread pool. Background tasks also do not scale and handle exceptions/retries poorly. For scale and reliability use specialised serving (Ray Serve, BentoML, vLLM) or a queue/broker stack (Celery + Redis + RabbitMQ).

## Concurrency strategy summary

Synchronous (single user, infrequent use); Async IO (blocking I/O work, fastest for I/O, one core/thread, easy to block by mistake); multithreading (I/O work, thread-safety hazards); multiprocessing (compute-bound, distributing across cores, but sharing a model between processes is hard).

## Latency vs throughput

Latency is time to first response; throughput is requests or tokens per minute. Larger models give higher quality but worse latency and throughput. Compression (quantization, pruning, distillation, small fine-tuned models) and transformer-inference tricks (fast attention, KV caching, paged attention, continuous batching) trade some of that back.

## Applying this when grading

Area 4:
- Are `async def` handlers free of blocking/sync calls? A sync call inside an async handler is the primary fail. A sync `def` handler is acceptable, not a fail.
- For hosted-provider services: is the provider's async client used, with backoff/throttling for rate limits?
- For self-hosted models: is serving externalised (vLLM/Ray/Triton or a separate server) rather than loaded in-process, given inference is memory-bound? Multiprocessing alone does not solve this.
- Are long-running jobs handled without blocking (background tasks for light work; external server/queue for heavy inference)? Flag heavy CPU-bound inference dumped into a background task expecting parallelism.
- Are timeouts, backoff, bounded concurrency, and resource cleanup present on external calls?
- Calibration: a simple or early-stage service being synchronous is acceptable; do not demand async heroics, demand the absence of event-loop blocking.

Area 3 (serving strategy):
- Is the serving strategy deliberate and matched to whether the model is hosted (async client) or self-hosted (externalised serving)? In-process loading of a large self-hosted model that blocks the server is a fail.
