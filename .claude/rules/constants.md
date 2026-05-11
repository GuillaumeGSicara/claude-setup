# Constants

## Overview

Project-wide constants belong **only** at the top level of the repo, inside a `constants.py file, or inside a dedicated directory `constants/` if there are many constants. This ensures they are easily discoverable and maintainable.
They serve as the single source of truth for values that never change at runtime (paths, hyper-parameters, external identifiers, etc.). They should be imported and used across the codebase instead of hardcoding values in multiple places.

## ✅ Correct conventions

| What | How |
|------|-----|
| **location** | All constants should be defined in a `constants.py` file at the root of the repo, or in a `constants/` directory if there are many constants. |
| **naming** | Use **ALL_CAPS_SNAKE_CASE** for constant names (e.g., `DATA_DIR`, `MAX_RETRIES`). |
| **typing** | Always include type annotations for constants, use `Final` when appropriate (e.g., `DATA_DIR: Final[str] = "/data"`). |
| **paths** | For file paths, use `pathlib.Path` objects instead of raw strings (e.g., `DATA_DIR: Path = Path("/data")`). |
| **immutability** | If a constant is a collection (e.g., list, dict), use immutable types like `tuple` or `frozenset` in addition to `Final` to prevent accidental modifications. |
| **single source** | Import the constant wherever it's needed instead of hardcoding values in multiple places. |


### Example

```python
from pathlib import Path
from typing import Final

DATA_DIR: Final[Path] = Path("/data")
MAX_RETRIES: Final[int] = 5
...
```

## ❌ Anti-patterns (never do these)

```python
# BAD — constants defined in multiple places
DATA_DIR = "/data"  # ❌ should be defined in a single `constants.py`
```

```python
...
if retry_count < 5:  # ❌ should use `MAX_RETRIES` constant instead of hardcoding `5`
    ...
```

```python
# some_module/some_file.py
from typing import Final
from pathlib import Path

DATA_DIR: Final[Path] = Path("/data")  # ❌ should be defined in a single `constants.py` file, not in multiple modules
```