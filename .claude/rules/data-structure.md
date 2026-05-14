## Data structure (Python)

## Overview

This document indicates the data structure conventions for the project. We favor simple, flat data structures (e.g. Pydantic BaseModel) over complex class hierarchies. We avoid inheritance and deep nesting, and prefer composition and explicit data modeling.


## ✅ Correct Conventions

### 1️⃣ Framework choice

Use Pydantic `BaseModel` for any data structure crossing a boundary: API requests/responses, configuration, LLM tool calls and outputs, queue messages, persisted records, anything serialized. Pydantic gives us validation, parsing, serialization, and schema generation for free at exactly the places we need them.

Plain `dataclass` (preferably `frozen=True, slots=True`) or ORM specific entities are fine for purely internal data structures that never cross a boundary. But as a rule of thumb, if you find yourself asking "should this be a Pydantic model?" the answer is probably yes.

The goal is to catch errors early at boundaries, have clear data contracts, and keep static typing and runtime validation aligned — without dragging Pydantic into places it doesn't earn its weight.

### 2️⃣ Avoid inheritance

Avoid inheritance and deep class hierarchies. Rule of thumb: if you find yourself over 2 levels deep, refactor. Instead, favor composition: build complex data structures by nesting simple `BaseModel` classes rather than inheriting from them.

The one legitimate use of inheritance here is a thin technical base model that captures a shared cross-cutting concern (e.g. a `TimestampedModel` with `created_at`/`updated_at`, or a `DomainEvent` base for an event bus). One level, no domain semantics in the base — anything else is a smell.

### 3️⃣ Explicit data modeling

Model your data explicitly with clear field definitions and types. Use `Field()` with a description whenever:
- the model is exposed in a schema (FastAPI/OpenAPI, MCP tool definition, JSON Schema for an LLM),
- the field has constraints worth enforcing (`gt`, `ge`, `min_length`, etc.),
- or the field name alone doesn't make the semantics obvious.

For self-evident fields like `user_id: UUID` or `created_at: datetime`, an annotation alone is fine — a ritual description that just restates the name is worse than no description.

Avoid implicit or dynamic attributes that are not defined in the model schema. Set `model_config = ConfigDict(extra="forbid")` on models receiving external data so unknown fields fail loudly rather than being silently dropped. Prefer `frozen=True` for value objects.

`extra="forbid"` is the right default on contracts *we* own (inbound API requests, internal messages, config) — unknown fields there are almost always bugs. It is the wrong default on payloads *we don't* own, typically third-party LLM, provider, or vendor responses: those evolve between versions, and a new field appearing in a provider release should not break production. Use `extra="ignore"` on those models, and lean on tests rather than the schema to catch shape drift.

### 4️⃣ Validation and defaults

Use Pydantic's validation features to enforce data integrity. Use `Field()` to specify defaults and single-field constraints directly in the model schema.

For more involved rules:
- **Single-field custom validation** → `Annotated[T, AfterValidator(fn)]`.
- **Cross-field invariants** (rules that depend on more than one field) → `@model_validator(mode="after")`.
- **Business rules that require external state** (DB lookups, remote calls, etc.) do *not* belong on the model. Keep them in the application layer; the model only validates what can be checked from the data itself.

```python
class Item(BaseModel):
    model_config = ConfigDict(extra="forbid", frozen=True)

    name: str = Field(..., description="Display name shown to end users")
    price_eur: float = Field(..., gt=0, description="Unit price in EUR; converted upstream if the source currency differs")
    quantity: int = Field(..., ge=0, description="Quantity of the item in stock")
    product_slug: Annotated[str, AfterValidator(ensure_slug_is_kebab_case)] = Field(
        ..., description="URL slug, unique identifier for the product"
    )
```

## ❌ Anti-Patterns

* Using `dataclass` or custom classes for data that crosses a boundary (API, config, LLM I/O, persistence).
```python
# BAD - dataclass for an API payload
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
```

* Deep inheritance hierarchies (e.g. `class A(BaseModel)`, `class B(A)`, `class C(B)`, etc.).
```python
# BAD - deep inheritance hierarchy
class A(BaseModel):
    id: int

class B(A):
    name: str

class C(B):
    email: str
```

* Implicit or dynamic attributes that are not defined in the model schema.
```python
# BAD - dynamic attributes not defined in the model
class User(BaseModel):
    id: int
    name: str

    def add_email(self, email: str):
        self.email = email  # not in the schema, won't be validated or serialized
```

* Missing `Field()` description where one is clearly warranted (schema-exposed model, non-obvious semantics, business-loaded field).
```python
# BAD - schema-exposed model with no descriptions
class CreateOrderRequest(BaseModel):
    amount: float
    currency: str # no description, is this the ISO code? a symbol? a full name? Could be an enum to clarify, or needs a description at minimum 
    customer_ref: str  # what format? internal id? external? a description belongs here
```

* Reinventing validation by writing a `.validate()` method that Pydantic doesn't know about.
```python
# BAD - validation method that nothing calls automatically
class Transaction(BaseModel):
    amount: float
    currency: str

    def validate(self):  # bypassed by normal Pydantic construction
        if self.amount <= 0:
            raise ValueError("Amount must be positive")
```
Use `Field(..., gt=0)`, `AfterValidator`, or `@model_validator` instead.