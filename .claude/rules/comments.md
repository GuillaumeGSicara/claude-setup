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