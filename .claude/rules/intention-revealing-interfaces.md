# Intention-Revealing Interfaces & Contracts

## Overview

A caller must understand a component solely from its signature (name, parameters, return type) without inspecting its implementation, preserving Information Hiding. Use documentation only to specify the formal Contract (preconditions, postconditions, and invariants) that the type system cannot natively enforce.

## Rule 1: Name operations after domain effects (What), not implementation mechanics (How)

**Principle: Intention-Revealing Interfaces (DDD) & Encapsulation**
Operations should be named using the project's Ubiquitous Language to expose the business goal, keeping technical implementation details hidden.

❌ Bad:

```python
def save_user_to_postgres_and_send_sns_message(data: dict):
```
(Leaks database and message queue details)

✅ Good:

```python
def register_new_member(member: NewMember):
```
(Exposes domain intention)

## Rule 2: Use one word per domain concept

**Principle: Self-Documenting Code & Ubiquitous Language**
Do not use different verbs for the same systemic action in different modules (e.g., mixing fetch, retrieve, and get for read-only lookups).

❌ Bad: `retrieve_invoice()` in one service, but `get_receipt()` in another doing the exact same type of resolution.
✅ Good: Standardize on a single domain verb: `resolve_invoice()` and `resolve_receipt()`.

## Rule 3 & 4: Name parameters to establish explicit Preconditions

**Principle: Design by Contract & Type-Safety.**
Parameters define the input constraints of your contract. Avoid generic types and names that hide domain invariants.

❌ Bad:

```python
def calculate_interest(amount: float, rate: float):
```
(Which currency? Is rate a fraction or percentage?)

✅ Good:

```python
def calculate_interest(principal: USD, annual_rate: Percentage):
```
(Clear type-driven preconditions)


## Rule 5: Respect Command-Query Separation (CQS)

**Principle: Principle of Least Surprise (POLA).**

A read-only operation (Query) must never modify state or trigger costly side-effects. A state-changing operation (Command) must explicitly signal its effect in its name.

❌ Bad:

```python
get_user_profile(user_id: int) -> Profile
```
(Hidden side-effect: silently fetches from a remote API and updates database cache)

✅ Good:

```python
fetch_and_cache_profile(user_id: int) -> Profile
```
(No astonishment—honors POLA)