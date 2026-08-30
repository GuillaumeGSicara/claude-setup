# Code Comments

## Overview

In this project we **avoid `#` comments that try to explain *what* the code does**.

Instead we rely on:
* **Expressive names** for functions, variables and classes.
* **Docstrings** for any higher-level explanations (module-level, class-level, or function-level).

This keeps the code self-documenting.


---

## ✅ Accepted conventions

### 1️⃣ Prefer expressive names

- Use descriptive names and type annotations to make the code self-explanatory.
```python
def load_agent_metadata_from_file(file_path: Path) -> AgentMetadata:
    ...
```
The function name and signature clearly indicate what the function does, so no additional comments are needed.

- Use docstrings for deeper context
```python
def extract_agent_name(metadata: AgentMetadata) -> str:
    """
    Extracts the agent's name from the metadata.

    The agent's name is determined by the following priority:
    1. If `metadata.name` is not empty, return it.
    2. Else if `metadata.id` is not empty, return it.
    3. Otherwise, return "Unknown Agent".
    """
    ...
```
Docstrings are searchable, show up in the IDE hover tips, and are the canonical way to provide explanations in Python.


- TODO and FIXME comments are acceptable
If a piece of code needs future work or a known issue needs to be addressed, it's acceptable to use `# TODO` or `# FIXME` comments to highlight these areas for future developers.

```python
# TODO: Implement the function to handle edge cases
# FIXME: This function currently does not handle empty metadata (see issue #123)
```

### 2️⃣ Docstrings document the contract, not the implementation

A docstring tells a caller what they need to know to use the function correctly — behavior, preconditions, postconditions, invariants. It is not the place for implementation history, design rationale, or a map of the surrounding code.

- **Never name another function, class, or module inline as justification.** If understanding this docstring requires jumping to another symbol, that reasoning belongs next to that symbol, not here — and naming a concrete implementation from an interface's docstring leaks it into the abstraction.

```python
# BAD — a Protocol method naming a concrete adapter
class FactResolver(Protocol):
    def research(self, question: str) -> ResearchedAnswer | None:
        """
        For a question where asking a search provider to name a value directly risks it silently
        picking a homonym's answer -- see `AnthropicSirenExtractor`, which reads the free text back
        out with a separate, narrow call instead of trusting a schema-fill step.
        """
        ...
```

```python
# GOOD — states the contract only
class FactResolver(Protocol):
    def research(self, question: str) -> ResearchedAnswer | None:
        """
        Answer `question` as free text, plus every source URL consulted, without structuring it.

        Prefer this over `resolve` when a direct schema-fill risks silently picking a homonym's
        answer; a caller can then extract the final value with its own narrower step.
        """
        ...
```

- **A single pointer to a design doc is fine; repeating its content is not.** When the contract alone would leave a genuinely non-obvious "why" unanswered, point to the one place that owns it (e.g. `docs/design/03-....md`) instead of narrating it in the docstring.
- **Decision-log entries don't belong in a docstring.** Sentences like "confirmed with the business on..." or "found through live testing that..." are commit-message or ADR content — a docstring that carries them will drift out of date in place instead of being superseded.

---

## Anti-patterns (never do these)

- Inline reporting of what the code does
```python
# Configuring page and checking if the user is authenticated
configure_page()

if not is_authenticated():
    st.warning("Please log in to access the dashboard.")
    st.stop()
```
The comment is redundant because the code is already clear and self-explanatory. 


- Comments indicating what a variable or constant does (see `constants.md` for how to properly document constants)
```python
# Max number of retry attempts for API calls
MAX_RETRIES: Final[int] = 5
```

- Long explanations of how the code works, that belong in docstrings instead
```python
# This function extracts the agent's name from the metadata. It first checks if the `name` field is not empty and returns it. If the `name` field is empty, it checks the `id` field and returns it if it's not empty. If both fields are empty, it returns "Unknown Agent". This logic ensures that we have a fallback mechanism for determining the agent's name, providing robustness in cases where certain metadata fields may be missing or incomplete.
def extract_agent_name(metadata: AgentMetadata) -> str:
    ...
```

- Large subdivision of orchestration code
```python
# ---------------------------------
# 1️⃣ Page Setup
# ---------------------------------
configure_page()
client_setup()

settings: ProjectSettings = load_settings()
```

- A docstring that narrates design history and cross-references other symbols instead of stating a contract
```python
def stamp_document(values, provenance_type, file_name, page=None):
    """
    Stamp every scalar, non-`MissingReason` field of `values` as coming from `file_name`, at `page`.

    Only touches fields whose provenance-mirror type is `Provenance | None` -- a nested model or a
    list is left at its mirror's own default (all-unresolved), for the caller to fill in
    explicitly. See e.g. `src.models.extraction_response.stamp_extraction_from_report`, which composes this for
    every model in the extraction contract, and `src.services.enrichment`, which overwrites the
    handful of entries a resolved source can outrank the report on.
    """
```
The last two sentences describe *other callers*, not this function's contract — delete them; a reader of `stamp_document` doesn't need to know who calls it to use it correctly.
