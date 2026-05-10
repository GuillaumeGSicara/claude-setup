# Typing

## Overview

In this project we **always use type annotations** to make code self-documenting, enable static type checking, and reduce cognitive load when reading the codebase.

## ✅ Correct conventions

| What | How |
|------|-----|
| **coverage** | Annotate **every** variable, parameter, and return — including `str`, `int`, `dict[str, str]`, `pd.DataFrame`. |
| **intent over shape** | Prefer domain types (`StrEnum`, `Literal`, `NewType`, `TypedDict`, `dataclass`, `pydantic.BaseModel`) over generic containers. |
| **structured data** | Model nested data with `pydantic.BaseModel` — never `dict[str, dict[str, ...]]`. |
| **avoid `Any`** | `Any` disables type checking. Confine it to I/O boundaries and narrow it immediately. |
| **dict depth** | Only **one-level-deep** dicts are allowed (`dict[str, int]`, `dict[str, MyModel]`). Anything deeper → use `pydantic.BaseModel`. |

### Example

```python
import json
from enum import StrEnum
from pathlib import Path
from typing import Any

import pandas as pd
from pydantic import BaseModel


class SupportedFileExtension(StrEnum):
    JSON = ".json"
    YAML = ".yaml"


class AgentConfiguration(BaseModel):
    name: str
    temperature: float
    tools: list[str]


def load_config(file_path: Path) -> AgentConfiguration:
    extension: SupportedFileExtension = SupportedFileExtension(file_path.suffix)

    with open(file_path, "r") as f:
        raw: dict[str, Any] = json.load(f)  # `Any` only at the I/O boundary

    config: AgentConfiguration = AgentConfiguration.model_validate(raw)
    return config


def average_score(experiment_results: pd.DataFrame) -> float:
    score: float = experiment_results["value"].mean()
    return score
```

## ❌ Anti-patterns (never do these)

```python
# BAD — missing annotations on "obvious" types
name = "claude"                    # ❌ should be `name: str = "claude"`
scores = {"a": 1, "b": 2}          # ❌ should be `scores: dict[str, int] = ...`
experiment_results = pd.read_csv("result_20210401.csv")       # ❌ should be `experiment_results: pd.DataFrame = ...`
```

```python
# BAD — `Any` leaks through the whole call chain, erasing all type safety
def process(data: Any) -> Any:
    return data["values"]
```

```python
# BAD — dicts deeper than one level are not allowed (unreadable, unvalidated)
user: dict[str, dict[str, str | int]] = {
    "profile": {"name": "alice", "age": 30},
    "address": {"city": "Paris", "zip": "75001"},
}

# GOOD — explicit pydantic models document the contract and validate input
class Profile(BaseModel):
    name: str
    age: int

class Address(BaseModel):
    city: str
    zip: str

class User(BaseModel):
    profile: Profile
    address: Address
```

```python
# BAD — vague signature, the contract is invisible
def evaluate(record: dict, threshold: float) -> dict: ...

# GOOD — self-documenting signature
def evaluate(record: UserRecord, threshold: float) -> EvaluationResult: ...
```