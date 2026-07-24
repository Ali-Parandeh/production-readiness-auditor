# O'Reilly fallback recommendations

Use this file only when the O'Reilly MCP is not connected. When the MCP is available, prefer live results so editions, links, and (where supported) actual book passages are current.

Recommend only for areas that scored **Partial** or **Fail**. Always include the primary reference first, with the chapter that matches the failed area.

## Primary reference (always include)

- **Building Generative AI Services with FastAPI** — Ali Parandeh (O'Reilly, 2025). The source this checklist is derived from. Map the failed area to its chapter:
  - Area 1 Modular architecture (Onion) -> Ch 2
  - Area 2 Routing and API design / type-safety -> Ch 2, Ch 4
  - Area 3 LLM integration and streaming -> Ch 3, Ch 5, Ch 6
  - Area 4 Async concurrency -> Ch 5
  - Area 5 Security and optimisation (auth) -> Ch 8, Ch 9, Ch 10
  - Area 6 Tests and behavioural testing -> Ch 11
  - Area 7 Docker and container security -> Ch 9, Ch 12

## Supporting reading by area

### Area 1 — Modular architecture (Onion)
- **Architecture Patterns with Python** — Harry Percival and Bob Gregory (O'Reilly). Repository pattern, dependency inversion, service layer, ports and adapters.

### Area 2 — Routing and API design
- **RESTful Web APIs** — Leonard Richardson, Mike Amundsen, Sam Ruby (O'Reilly). Resource modelling, status codes, API design that maps onto FastAPI routing.

### Area 3 — LLM integration and streaming
- **AI Engineering** — Chip Huyen (O'Reilly, 2025). Serving model-backed applications and the surrounding production concerns.

### Area 4 — Async concurrency
- **Using Asyncio in Python** — Caleb Hattingh (O'Reilly). The event loop, coroutines, and avoiding blocking calls.
- **Fluent Python** — Luciano Ramalho (O'Reilly). Concurrency and async chapters for the mechanics behind the GIL trade-offs.

### Area 5 — Security and optimisation
- **AI Engineering** — Chip Huyen (O'Reilly, 2025). Guardrails, evaluation, and inference optimisation including caching.

### Area 6 — Tests and behavioural testing
- **Python Testing with pytest** — Brian Okken. Fixtures, mocking, and a maintainable suite (on the O'Reilly platform).
- For behavioural testing of model outputs, see the CheckList method (Ribeiro et al., "Beyond Accuracy: Behavioral Testing of NLP Models with CheckList", ACL 2020): MFTs, ITs, and DETs.

### Area 7 — Docker and container security
- **Docker: Up & Running** — Sean Kane and Karl Matthias (O'Reilly). Multi-stage builds, runtime configuration.
- **Container Security** — Liz Rice (O'Reilly). Non-root users and image hardening.

---

Note: verify titles, editions, and availability through the live MCP when possible. This is a curated starting point, not an exhaustive catalogue, and is meant to be refined.
