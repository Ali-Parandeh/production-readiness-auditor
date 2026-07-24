# Chapter 2 — Project structure and the onion design pattern

Grounding for **Area 1 (modular architecture)** and the design parts of **Area 2 (routing and API design)**. Read this before grading those areas so feedback uses the book's own framework, not generic FastAPI advice. Source: *Building Generative AI Services with FastAPI*, Ch 2.

## Project structure: flat → nested → modular

The book's position: pick the structure that fits current complexity, and **reorganise progressively** as the service grows. There is no single correct layout; the test is whether you can justify it.

**Flat** — all application files at the root of one `app` directory, grouped by kind:
```
flat-project/app/{main.py, routers.py, services.py, models.py, database.py}
```
Right for a first version or a tiny microservice. You can ignore coupling and reuse because there is little code. Becomes hard to maintain as complexity grows.

**Nested** — similar modules grouped into packages by kind:
```
nested-project/app/{main.py, dependencies.py}
  services/{users.py, profiles.py}
  models/{users.py, profiles.py}
  routers/{users.py, profiles.py}
```
Recommended by the official FastAPI docs for larger projects. This is an AI microservice. Pitfall: **ambiguous coupling** between packages, where a change cascades into many files (the book calls this *shotgun updates*).

**Modular** — closely related modules grouped by **domain/feature**, not by kind (popularised by Netflix Dispatch):
```
modular-project/app/
  modules/
    auth/{routers.py, models.py, dependencies.py, guards.py, services.py}
    users/{router.py, models.py, dependencies.py, services.py, mappers.py, pipes.py}
    profiles/...
  routers/users.py
  providers/{email.py, stripe.py}
  settings.py        # global config
  middlewares.py     # global middleware
  models.py          # global models
  exceptions.py      # global exceptions
  main.py
```
Right for a full backend service serving an AI model alongside auth, external systems, and complex logic. Encapsulation removes uncertainty about coupling and improves scalability and maintainability.

**Progression rule of thumb:** flat = experimenting / first version; nested = AI microservice; modular = full backend service.

**Heuristic to apply when grading:** if the developer cannot justify the file organisation to another developer, the structure needs reconsidering. A complex GenAI backend still sitting in a flat layout is the most common mismatch.

## Onion / layered design pattern

Sits inside the nested or modular structure. Purpose: separation of concerns so features are easy to add, remove, and change. Layers build on each other with the **domain model and business logic at the centre**, and every outer layer depends **inward**.

Core principle: **dependency inversion**. High-level modules do not depend on the implementation of low-level modules. They declare what they need and let FastAPI's dependency system inject it, which keeps layers decoupled.

Layers, outer to inner:

- **API routers** — group route handlers and apply common logic across them, via `APIRouter`.
- **Controllers / route handlers** — handle incoming requests and return responses by orchestrating services/providers. Good controllers inject what they need through dependencies rather than constructing it inline.
- **Services and providers** — services orchestrate internal operations to implement business logic and use repositories for data access; **providers** are the same idea but specialised for *external* systems (email servers, payment gateways, other microservices). The services-vs-providers split is a book-specific distinction worth checking.
- **Repositories (data adapters)** — encapsulate data access and mutation via ORM or raw SQL. May expose an abstract CRUD interface for consistency across repositories.
- **Schemas / models** — enforce type-safety, structure, and validation as data flows through the service.

Components that span layers:

- **Middleware** — processes requests/responses before and after the controllers.
- **Dependencies** — reusable injectable functions; can be cached and can depend on other dependencies.
- **Pipes** — data transformers used across layers (aggregators, cleaners, parsers, translators).
- **Mappers** — convert one schema into another across layers (for example `UserRequest` at the router layer into `UserInDB` at the data layer).
- **Exception filters** — handle exceptions consistently across layers. This is what grounds the "consistent error envelope" check in Area 2.
- **Guards** — protect controllers from abuse; auth and authorisation implemented as dependencies or middleware. (Audited mainly in Area 5.)

## Applying this when grading

Area 1:
- Identify the structure (flat / nested / modular) and judge it against the project's actual complexity. Flag a complex GenAI service still in a flat layout.
- Check the dependency direction is inward: the domain/business logic must not import the web framework or data-access detail.
- Check the onion layers are present and separated: routers, controllers, services, repositories, schemas. Note whether **services and providers** are distinguished, or whether external-system calls are mixed into services.
- Check cross-cutting pieces exist where the app needs them: middleware, shared dependencies, exception filters, guards.
- Use the "justify it to another developer" heuristic for borderline cases.

Area 2:
- Controllers inject data and logic via `Depends` rather than building it inline.
- `APIRouter` used to group routes and apply shared logic.
- Schemas/models enforce validation and type-safety (carry into Ch 4 type-safety when grading).
- Errors handled through exception filters, not ad hoc dicts per route.
