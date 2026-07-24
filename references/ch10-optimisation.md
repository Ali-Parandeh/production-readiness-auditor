# Chapter 10 — Performance and quality optimisation

Grounding for the optimisation part of **Area 5**. Source: *Building Generative AI Services with FastAPI*, Ch 10.

Optimisation targets either **performance** (latency, throughput, cost) or **output quality**. None of it is mandatory; apply each where it earns its place.

## Batch processing

Naively making one API call per entry is slow, costly, and invites rate limiting. Two better techniques:
- **Structured-output batching:** update the Pydantic schema or prompt template to request a **list** of outputs per request, so many entries are processed in a handful of calls instead of one each.
- **Provider batch APIs** (for example OpenAI's batch API): submit async groups of requests via a JSONL file (one request per line). Lower cost (up to ~50%), higher rate limits, guaranteed completion times. Ideal for jobs that do not need an immediate response (bulk parse, classify, translate). Runs as a background task queue with status updates.

## Caching

Cache frequently accessed or expensive results to cut latency, server load, and cost. Match the refresh/invalidation frequency to acceptable staleness. Three strategies, chosen by how variable the inputs are:

- **Keyword caching:** exact-match key-value. `fastapi-cache` implements it on functions/endpoints in a few lines, with a Redis backend to share the cache across instances. Sets `Cache-Control` directives (`max-age`, `no-cache`, `no-store`, `private`). Suits repeated identical inputs; poor for variable user queries.
- **Semantic caching:** returns a stored value for **semantically similar** inputs, using embeddings and a similarity search over stored pairs. Cuts API calls to 30-40% (a 60-70% hit rate); a Q&A RAG app can cut calls by ~69%. Two cache points in a RAG system: before the LLM (return a cached response) and before the vector store (reuse retrieved documents). Build from scratch with a cache store client, a vector store client, an embedding model, and a tuned **similarity threshold** (above it is a hit), or use `gptcache`.
  - **UX caveat:** caching LLM responses can backfire when similar prompts should produce different output, for example "summarise in 100 words" vs "summarise in 50 words" collapse to the same cached answer. Prefer caching **document retrieval** over final responses where the output should vary.
  - Tune the threshold: too high gives few hits, too low gives false positives.
  - **Eviction policies:** FIFO, LRU, LFU, MRU, RR. Start with LRU.
- **Context / prompt caching:** reuses precomputed attention states for a large context referenced repeatedly across small requests, avoiding recomputation. Use for chatbots with long system instructions and multiturn history, recurring queries over large document sets, repeated code-repo analysis, long-form summarisation, and many in-context examples. Large savings on long prompts (Anthropic cites up to 90% cost and 85% latency). Note it introduces **statefulness** across requests. OSS option: MemServe (avoids vendor lock-in).

## Model quantization

Relevant only if you **self-host** models. Quantization projects high-precision weights/activations into lower precision, cutting memory and energy and speeding up integer arithmetic, at some quality cost. Pre-quantized versions are often available to download. Not applicable to services that only call a hosted provider.

## Prompt engineering (quality)

Refining prompts to improve output quality without fine-tuning or retraining. Treat it as a communication problem: the model is a knowledgeable colleague that needs clear, specific instructions and examples; vague prompts give average output. It can be approached with TDD, refining prompts until behavioural tests pass.

Use a systematic system-prompt template, the book's **RCT** template:
- **Role:** how the model should behave (roles measurably change outputs; a persona adds more).
- **Context:** the scenario and reference information; in RAG, the system prompt plus retrieved chunks.
- **Task:** clear, unambiguous instructions, ideally with a few examples.

Advanced families (from a systematic survey): in-context learning, thought generation, decomposition, ensembling, self-criticism, agentic.

## Grading calibration (Area 5, optimisation)

- Optimisation is value-driven. Only flag missing caching or batching where the traffic is repetitive or high enough to pay off; do not penalise a low-traffic service.
- Match the caching strategy to the input pattern: keyword for exact repeats, semantic for similar-meaning queries, context/prompt for large repeated context. Flag semantic caching of responses where outputs ought to vary.
- Quantization only applies when self-hosting; ignore it for hosted-API services.
- For quality, a structured system prompt (RCT or equivalent) is the low-effort baseline; ad hoc prompts scattered in code are a weakness.
