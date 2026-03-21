# Streamlit Session State Rules

## Rule: Always use `StateHandler` — never access `st.session_state` directly

All reads and writes to Streamlit session state **must** go through `StateHandler` from `state.py`. Direct use of `st.session_state` (get, set, or check) is forbidden in pages and controllers.

### Why

`StateHandler` is a typed wrapper around `st.session_state` that:
- Enforces a declared schema (undeclared fields raise `AttributeError`)
- Keeps `st.session_state` and the handler in sync automatically via `__setattr__`
- Provides IDE-friendly typed access to all state fields

---

## Usage patterns

### In a page (`pages/<name>.py`)

Instantiate `StateHandler` once at the top of the page, then read/write only through it.

```python
from state import StateHandler

state = StateHandler()

# Read
company = state.selected_company

# Write
state.product_history = get_product_history(...)
```

### In a controller

Accept `state: StateHandler` as a parameter; do not instantiate a new one inside the controller.

```python
from state import StateHandler

def redirect_to_home(state: StateHandler) -> None:
    state.selected_company = None
    st.switch_page(HOME_VIEW)
```

---

## Declaring new state fields

Add the field to `StateHandler` and provide its default in `_get_default_value`:

```python
class StateHandler:
    settings: ProjectSettings
    env: Environment
    selected_company: Company | None   # new field

    def _get_default_value(self, name: str) -> Any:
        ...
        if name == "selected_company":
            return None
```

---

## Anti-patterns (never do these)

```python
# BAD — direct read
value = st.session_state["my_key"]
value = st.session_state.get("my_key")

# BAD — direct write
st.session_state["my_key"] = value
st.session_state.my_key = value

# BAD — instantiating StateHandler inside a controller
def my_controller():
    state = StateHandler()   # controllers receive state as a parameter
    ...
```
