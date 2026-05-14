# N-Tier architecture for APIs

## Overview

This project follows a **layered (n-tier) architecture** for APIs, with a clear separation of concerns across tiers. This promotes modularity, testability, and maintainability, and makes the codebase easier to scale and refactor.

The layering described here is **closed** (a.k.a. strict): each layer depends only on the layer directly below it, and never on the layers above. Shared cross-cutting modules (models, exceptions, settings, configuration) are explicitly called out below — they are not layers in the dependency stack.

## ✅ Conventions

### 1️⃣ Layered structure

We follow the standard **4-tier layered model**:

| Tier | Role | Python module | Java package |
|---|---|---|---|
| **Presentation** | HTTP entrypoints: routing, request/response serialization, status codes, OpenAPI contract. No business logic. | `api/` | `controller/` |
| **Application** | Orchestration of use cases: coordinates domain operations, transactions, and calls to external resources. No HTTP, no SQL. | `services/` | `service/` |
| **Domain** | Core business rules and invariants. Pure logic; framework- and IO-free. | `domain/` | *(business logic inside `service/` or a dedicated domain package)* |
| **Data Access** | Persistence: SQL, ORM, external store clients. Exposes repository interfaces to the application layer. | `repositories/` | `repository/` |

Dependency direction is always **top → down**. The presentation layer never imports from the data access layer; the application layer never imports from the presentation layer.

> 💡 The **Domain** tier is optional for small APIs. In that case, business rules live inside the application layer (`services/`). Once domain logic grows beyond simple orchestration, extract it into a dedicated `domain/` package. Do not pre-emptively create empty layers.

> ⚠️ Terminology note: in the FastAPI/Spring world, **"controller"** is overloaded. In Java/Spring it conventionally means the HTTP entrypoint (`@RestController`). In some Python codebases it has been used for business logic. To avoid this conflict, we **do not use `controller/` in Python** — use `api/` for HTTP entrypoints and `services/` for orchestration.

#### Python API structure

```shell
src/
├── api/                # Presentation: FastAPI routers, request handling
├── services/           # Application: orchestration of use cases
├── domain/             # Domain: business rules (optional for small APIs)
├── repositories/       # Data Access: persistence and external store clients
├── models/             # Domain models (shared, see §2)
├── schemas/            # API contracts: Pydantic request/response DTOs (see §2)
├── exceptions/         # Custom exceptions (shared)
├── settings/           # Application configuration via pydantic-settings (shared)
├── configuration/      # Global setup: logging, DB connections, DI wiring (shared)
tests/              # Mirrors the layer structure (see §5)
```

#### Java API structure

```shell
module/
├── controller/         # Presentation: @RestController, @RequestMapping
├── service/            # Application + Domain: business workflows and rules
├── repository/         # Data Access: JpaRepository interfaces
├── model/              # JPA entities (shared)
├── dto/                # API contracts with validation annotations (shared)
├── mapper/             # MapStruct interfaces between dto ↔ model
├── exception/          # Custom exceptions (shared)
└── validator/          # Business validation (optional)
```

The Python and Java structures map cleanly: `api/` ↔ `controller/`, `services/` (+ optional `domain/`) ↔ `service/`, `repositories/` ↔ `repository/`, `schemas/` ↔ `dto/`, `models/` ↔ `model/`.

### 2️⃣ Shared cross-cutting modules

Some modules are used by multiple layers and do not belong to the tier stack. They must not depend on any layer; all layers may depend on them.

* **`models/`** — Domain or persistence entities. Plain data structures (Pydantic, dataclass, or ORM entities). No business logic, no IO.
* **`schemas/`** (Python) / **`dto/`** (Java) — API contracts. Pydantic models / annotated DTOs used at the presentation boundary. **Keep these distinct from `models/`** — never return ORM entities directly from HTTP handlers. The mapping happens in the presentation or application layer (Python) or through `mapper/` (Java).
* **`exceptions/`** — Custom exception hierarchy, typically a domain exception base class plus specific subclasses. Layers raise; presentation translates to HTTP status.
* **`settings/`** — Configuration loaded from env / `.env` via `pydantic-settings` (Python) or `@ConfigurationProperties` (Java).
* **`configuration/`** — Wiring: logging setup, DB engine/session, DI container, FastAPI app/dependency providers. This is *infrastructure setup*, distinct from runtime layers.

> 💡 settings/, configuration/ and exceptions/, these start as single files and graduate to directories once they grow past ~1000 lines or accumulate several distinct concerns.

> 💡 The list of shared modules is intentionally limited. Adding a new one is a deliberate architectural decision — discuss with the user/architect before introducing one.

### 3️⃣ Inter-layer interfaces

Each layer exposes its capabilities through explicit, typed interfaces. The adjacent layer depends on the interface, not on the implementation.

In Python, use:
* **`typing.Protocol`** for duck-typed interfaces (preferred for repositories, external clients).
* **`abc.ABC`** when a real inheritance hierarchy is needed.
* **Pydantic models** (or frozen dataclasses) for data crossing layer boundaries.

In Java, use interfaces (`JpaRepository`, `@Service`-annotated interfaces) and DTOs with validation annotations.

Dependency injection (FastAPI `Depends`, Spring `@Autowired`) wires concrete implementations to interfaces at the composition root (`configuration/` in Python, `@Configuration` classes in Java).

### 4️⃣ Enforcing the architecture

In Python, use [import-linter](https://import-linter.readthedocs.io/) to enforce layer dependency rules.

* **Configure rules at the package level**, not per-file, to keep maintenance bounded.
* **Verify the current import-linter contract syntax against upstream docs** (`https://import-linter.readthedocs.io/`) before writing the configuration — the contract types and options evolve. Use web fetch / context7 / official docs, whichever is available.
* **Tune strictness to the project size.** Lightweight projects may want only a `layers` contract; larger projects benefit from additional `forbidden` and `independence` contracts. Discuss the trade-off with the user before committing to a strict ruleset.

In Java, layer enforcement typically uses [ArchUnit](https://www.archunit.org/) with similar package-level rules.

### 5️⃣ Tests

Tests mirror the layer they cover:
They should be placed at the repository root in a `tests/` directory, with subdirectories for unit, integration, and end-to-end tests. Unit tests are further organized by layer, while integration and e2e tests may be organized by feature or flow.

```shell
tests/
├── unit/
│   ├── domain/         # Pure business rules, no IO
│   ├── services/       # Application orchestration with mocked repositories
│   └── repositories/   # Repository logic with in-memory or test DB
├── integration/        # Cross-layer flows with real DB
└── e2e/                # HTTP-level tests against the API
```

The cross-layer dependency direction applies to tests too: unit tests for layer N may import from layers below, never above.

## ❌ Anti-patterns

* **Skipping layers.** Presentation reaching directly into the data access layer (e.g., a FastAPI route doing SQLAlchemy queries). Always go through the application layer.
* **Mixing concerns within a layer.** Business rules in the presentation layer, HTTP concerns in the application layer, or SQL fragments in the domain layer.
* **Leaking persistence models through HTTP.** Returning ORM entities (Python `models/`, Java `model/`) directly from API responses. Use `schemas/` / `dto/`.
* **Implicit interfaces.** Layers depending on concrete classes instead of `Protocol` / ABC / interface, making swapping implementations and mocking in tests painful.
* **Cross-cutting bloat.** Treating `configuration/` or `exceptions/` as a dumping ground for anything that doesn't fit elsewhere. If a module isn't clearly cross-cutting, it belongs in a layer.
* **Terminology drift.** Reusing layer names with different meanings across the codebase (e.g., `controller/` meaning HTTP in one module and business logic in another). Stick to the table in §1.

> 💡 This layered model can be combined coherently with patterns like Hexagonal/Ports-and-Adapters or DDD — the application layer maps to use cases, repositories map to ports, and the domain layer holds the model. What to avoid is **mixing the terminology incoherently** within a single codebase. Pick a vocabulary and stick to it.