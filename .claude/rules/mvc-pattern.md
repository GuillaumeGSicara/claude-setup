# Model-View-Controller (MVC) Pattern in Streamlit

## Overview

This project uses a strict MVC separation across four layers:

| Layer | Location | Responsibility |
|-------|----------|----------------|
| **Models** | `app/models/` | Data shapes and business logic (Pydantic) |
| **Views** | `app/views/` | Rendering only — no data fetching or state mutation |
| **Controllers** | `app/controller/` | Data fetching, transformation, and state transitions |
| **Infrastructure** | `app/infra/` | External system access (DB, APIs, caches) |

Pages (`app/pages/`) are thin orchestrators: they instantiate state, call controllers, and pass results to views.

---

## Layer responsibilities

### Models (`app/models/`)

Pure data containers and domain logic. Use Pydantic `BaseModel` for structured data.

- No Streamlit imports
- No database or API calls
- May contain display helpers (e.g. `get_display_text()`)

```python
# models/item.py
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    score: float

    def get_label(self) -> str:
        return f"{self.name} ({self.score:.0%})"

class ItemList(BaseModel):
    items: list[Item] = []
```

### Infrastructure (`app/infra/`)

Raw access to external systems (databases, APIs). Returns primitive types or DataFrames — no domain models.

- No Streamlit imports (except for caching, or pure streamlit backend related tasks if needed)
- No business logic
- Functions accept explicit connection/session objects

```python
# infra/db.py
import pandas as pd
from snowflake.snowpark import Session

def fetch_items(session: Session, category: str) -> pd.DataFrame:
    return session.sql(f"SELECT * FROM items WHERE category = '{category}'").to_pandas()
```

### Controllers (`app/controller/`)

Orchestrate infra calls, apply business logic, and return domain models. Use `@st.cache_data` for expensive operations.

- Accept `state: StateHandler` for state reads/writes, or explicit typed parameters
- Underscore-prefix non-serialisable args to opt out of Streamlit's cache key (e.g. `_session`)
- Return models, never render anything

```python
# controller/items.py
import streamlit as st
from infra.db import fetch_items
from models.item import Item, ItemList
from snowflake.snowpark import Session

@st.cache_data
def get_items(_session: Session, category: str) -> ItemList:
    df = fetch_items(_session, category)
    return ItemList(items=[Item(name=row["name"], score=row["score"]) for _, row in df.iterrows()])
```

State-mutating controllers (navigation, form submission) accept `state: StateHandler`:

```python
# controller/navigation.py
import streamlit as st
from state import StateHandler
from constants import HOME_VIEW

def go_home(state: StateHandler) -> None:
    state.selected_item = None
    st.switch_page(HOME_VIEW)
```

### Views (`app/views/`)

Pure rendering functions. Accept models as arguments, call only Streamlit display functions.

- No data fetching
- No state reads or writes
- No business logic

```python
# views/items.py
import streamlit as st
from models.item import ItemList

def render_item_list(items: ItemList) -> None:
    for item in items.items:
        st.write(item.get_label())
```

### Pages (`app/pages/`)

Thin orchestrators. Follow this order:
1. Configure page
2. Auth guard (`st.session_state.get("authenticated")` only here)
3. Instantiate `StateHandler`
4. Call controllers
5. Call views

```python
# pages/items.py
import streamlit as st
from state import StateHandler
from controller.items import get_items
from views.items import render_item_list
from utils.streamlit_config import configure_page
from constants import LOGIN_VIEW

configure_page("Items")

if not st.session_state.get("authenticated"):
    st.switch_page(LOGIN_VIEW)

state = StateHandler()

items = get_items(_session=state.session, category="tools")
render_item_list(items)
```

---

## Anti-patterns (never do these)

```python
# BAD — data fetching inside a view
def render_items() -> None:
    df = session.sql("SELECT ...").to_pandas()  # views never fetch data
    st.write(df)

# BAD — Streamlit calls inside a controller
def get_items(_session: Session) -> ItemList:
    st.spinner("Loading...")  # controllers never render
    ...

# BAD — business logic inside a page
state = StateHandler()
items = [i for i in raw if i["score"] > 0.5]  # belongs in controller or model

# BAD — infra call directly from a page
df = session.sql("SELECT ...").to_pandas()  # always go through a controller

# BAD — model contains a DB call
class Item(BaseModel):
    @classmethod
    def from_db(cls, session: Session) -> list["Item"]:  # belongs in infra + controller
        ...

# BAD — controller instantiates StateHandler
def go_home() -> None:
    state = StateHandler()  # receive state as a parameter instead
    state.selected_item = None
```
