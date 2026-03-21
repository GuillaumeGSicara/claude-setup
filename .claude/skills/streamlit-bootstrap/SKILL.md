---
name: streamlit-bootstrap
description: Scaffold a new Streamlit application following specifics conventions. Use when the user asks to "bootstrap a streamlit app", "scaffold a new streamlit project", "create a streamlit app structure", or "initialize a streamlit app".
disable-model-invocation: true
---

# Streamlit Bootstrap Skill

Scaffold a new Streamlit app following the conventions of this repository.

---

## WORKFLOW

1. Run **DIRECTORY_GUARD**
2. Run **EXISTING_FILE_GUARD**
3. Run **PROJECT_GUARD**
3. Run **PROCESS** (ask questions, generate files)
4. Run **VERIFICATION (AUTOMATIC)**
5. Print **NEXT STEP** message

---

## DIRECTORY_GUARD

Check the current working directory before doing anything else:

- If no `pyproject.toml` exists in cwd → stop and say:
  > "No `pyproject.toml` found. Please run this skill from a valid project root"
  use `AskUserQuestion` to ask:
    - "Would you like to create a python project base with the skill `python-project-init` first?"
      1. Yes → run `python-project-init` skill, then restart this skill
      2. No → stop — print "Aborted. No files were changed."
- If an `app/` directory already exists → skip to **EXISTING_FILE_GUARD**

---

## EXISTING_FILE_GUARD

Check which of these paths already exist:
- `app/app.py`, `app/state.py`, `app/settings.py`, `app/constants.py`, `app/config.py`
- `app/.streamlit/config.toml`
- `app/pages/`, `app/views/`, `app/models/`, `app/controller/`, `app/infra/`, `app/utils/`, `app/static/`
- `.env.template`

If **any** exist, use `AskUserQuestion` to ask:

> "Some files/directories that would be created already exist:
> {list them}
>
> How would you like to proceed?
> 1. Backup existing files (rename to .bak) then overwrite
> 2. Overwrite without backup
> 3. Abort"

- If user chooses **1**: rename each existing file/dir by appending `.bak` suffix, then continue
- If user chooses **2**: continue (overwrite)
- If user chooses **3**: stop — print "Aborted. No files were changed."

---

## PROJECT_GUARD

if the following packages are not in the pyproject.toml dependencies, add them by running `uv add` for each:
- streamlit
- pydantic
- pydantic-settings

do the same for dev-dependencies by running `uv add --dev` for each:
- pytest
- mypy
- ruff
- toml
- pre-commit
- xenon
- radon[toml]

---

## PROCESS

Ask the following questions **one at a time** using `AskUserQuestion`. Wait for each answer before asking the next.

### Question 1 — App name
> "What is the app name? (used in pyproject.toml `name` field and loading screen, e.g. `my-app`)"

Store as: `APP_NAME`

### Question 2 — Display title
> "What is the display title shown in the Streamlit UI heading? (e.g. `My App`)"

Store as: `DISPLAY_TITLE`

### Question 3 — Brand subtitle
> "What is the brand/subtitle shown below the title on the loading screen? (e.g. `Theodo`)"

Store as: `BRAND_SUBTITLE`

### Question 4 — First page name
> "What is the name of your first page? (snake_case, e.g. `dashboard`, `selection`)"

Store as: `FIRST_PAGE` (always lower snake_case)

### Question 5 — Colors
> "Primary color and background color? (press Enter to use defaults: primary=`#bb5a38`, background=`#f4f3ed`)"

Parse the answer:
- If blank → use defaults `#bb5a38` and `#f4f3ed`
- If user provides two hex codes (e.g. `#ff0000 #ffffff`) → use those

Store as: `PRIMARY_COLOR`, `BG_COLOR`

---

After collecting all answers, generate the files below. Replace every placeholder literally:
- `{{APP_NAME}}` → value of APP_NAME
- `{{DISPLAY_TITLE}}` → value of DISPLAY_TITLE
- `{{BRAND_SUBTITLE}}` → value of BRAND_SUBTITLE
- `{{FIRST_PAGE}}` → value of FIRST_PAGE
- `{{PRIMARY_COLOR}}` → value of PRIMARY_COLOR
- `{{BG_COLOR}}` → value of BG_COLOR

Derive these automatically:
- `{{FIRST_PAGE_UPPER}}` = FIRST_PAGE in UPPER_CASE (e.g. `DASHBOARD`)

---

### File: `app/__init__.py`
```python
```
*(empty file)*

---

### File: `app/app.py`
```python
"""Main application entry point"""

import time

import streamlit as st
from utils.css_loader import inject_css
from utils.streamlit_config import configure_page
from constants import CSS_DIR, {{FIRST_PAGE_UPPER}}_VIEW

# Page configuration
configure_page("{{DISPLAY_TITLE}}")

# Load CSS styles
inject_css(CSS_DIR / "app.css")

# Centered loading screen
st.markdown(
    """
    <div class="center-container">
        <div class="app-title">{{DISPLAY_TITLE}}</div>
        <div class="app-subtitle">{{BRAND_SUBTITLE}}</div>
    </div>
    """,
    unsafe_allow_html=True,
)

# Loading with spinner
with st.spinner(""):
    time.sleep(1.5)

# Redirect to first page
st.switch_page({{FIRST_PAGE_UPPER}}_VIEW)
```

---

### File: `app/state.py`
```python
"""Module to handle streamlit state"""

from typing import Any

import streamlit as st
from settings import Environment, ProjectSettings


class StateHandler:
    """
    Thin wrapper around st.session_state for convenient access and initialisation.
    List member(s) of Session state below; default values will be used by default.
    """

    # State fields
    settings: ProjectSettings
    env: Environment

    def __init__(self) -> None:
        for name, _ in self.__class__.__annotations__.items():
            if name in st.session_state:
                setattr(self, name, getattr(st.session_state, name))
            else:
                default_value = self._get_default_value(name)
                setattr(self, name, default_value)

    def _get_default_value(self, name: str) -> Any:
        """Get default value for a state field."""
        if name == "settings":
            return ProjectSettings()  # type: ignore
        if name == "env":
            return self.settings.environment
        else:
            raise AttributeError(f"No default value defined for {name}")

    def __setattr__(self, name: Any, value: Any) -> None:
        """Sync st.session_state and prevent undeclared fields."""
        if name not in self.__class__.__annotations__:
            raise AttributeError(f"{name} is not a valid state field, cannot mutate it")

        setattr(st.session_state, name, value)
        super().__setattr__(name, value)
```

---

### File: `app/settings.py`
```python
from enum import Enum

from pydantic import Field
from pydantic_settings import BaseSettings, SettingsConfigDict

from constants import PROJECT_ROOT


class Environment(str, Enum):
    """Runtime environment for the application."""

    PRD = "prd"
    HPRD = "hprd"
    LOCAL = "local"

    def get_coloured_markdown_string(self) -> str:
        match self.value:
            case Environment.LOCAL:
                return "LOCAL"
            case Environment.HPRD:
                return "HPRD"
            case Environment.PRD:
                return "PRD"
            case _:
                raise ValueError(f"{self.value} environment is unknown")


class ProjectSettings(BaseSettings):
    """Application settings loaded from environment variables.

    The pydantic-settings library reads variables from .env and loads them
    into the object AND into your environment variables.

    Warning: environment variables have precedence over .env file values.
    If you modify .env, relaunch your shell or source .env.
    """

    model_config = SettingsConfigDict(
        env_file=PROJECT_ROOT / ".env", env_file_encoding="utf-8", extra="ignore"
    )

    environment: Environment = Field(
        description="Runtime environment for the application"
    )
```

---

### File: `app/constants.py`
```python
from pathlib import Path

# Project root
PROJECT_ROOT = Path(__file__).parent.parent

# Streamlit application paths
SIS_ROOT = Path(__file__).parent
CSS_DIR = SIS_ROOT / "static"

# Page view paths
{{FIRST_PAGE_UPPER}}_VIEW: Path = PROJECT_ROOT / "app" / "pages" / "{{FIRST_PAGE}}.py"


# UI Labels
class UILabels:
    """UI text labels.

    Centralizing all UI labels makes it easier to:
    - Maintain consistency across the application
    - Add multi-language support in the future
    - Update terminology in one place
    """

    # Page titles
    HOME = "{{DISPLAY_TITLE}}"

    # Actions and navigation
    # TODO: add labels as you build pages
```

---

### File: `app/config.py`
```python
"""Application configuration constants.

This module centralizes all configuration values to:
- Make them easy to find and modify
- Avoid hardcoding values throughout the codebase
- Clearly separate configuration from business logic
"""

# UI theme constants (design tokens)
COLORS = {
    "primary": "{{PRIMARY_COLOR}}",
    "background": "{{BG_COLOR}}",
}
"""Color palette used throughout the UI"""

FONT_SIZES = {
    "xs": "0.75rem",   # 12px
    "sm": "0.875rem",  # 14px
    "base": "1rem",    # 16px
    "lg": "1.125rem",  # 18px
    "xl": "1.25rem",   # 20px
}
"""Font size scale for consistent typography"""

# Business logic thresholds
# TODO: add domain-specific thresholds here
```

---

### File: `app/.streamlit/config.toml`
```toml
[server]
enableStaticServing = true

[client]
showSidebarNavigation = false

[theme]
primaryColor = "{{PRIMARY_COLOR}}"
backgroundColor = "{{BG_COLOR}}"
secondaryBackgroundColor = "#ecebe3"
textColor = "#3d3a2a"
linkColor = "#3d3a2a"
borderColor = "#d3d2ca"
showWidgetBorder = true
baseRadius = "0.6rem"
font = "sans serif"
codeFont = "monospace"
showSidebarBorder = true
headingFontSizes = ["40px", "32px", "24px"]
headingFontWeights = 500

[theme.sidebar]
backgroundColor = "#e8e7dd"
secondaryBackgroundColor = "#ecebe3"
```

---

### File: `app/pages/__init__.py`
```python
```
*(empty file)*

---

### File: `app/pages/{{FIRST_PAGE}}.py`
```python
"""{{DISPLAY_TITLE}} — {{FIRST_PAGE}} page"""

import streamlit as st
from utils.css_loader import inject_css
from utils.streamlit_config import configure_page
from constants import CSS_DIR

# Page configuration — must be first Streamlit command
configure_page("{{DISPLAY_TITLE}}")

# Load CSS styles
inject_css(CSS_DIR / "app.css")

# TODO: implement {{FIRST_PAGE}} page
st.title("{{DISPLAY_TITLE}}")
st.write("Welcome! Add your content here.")
```

---

### File: `app/views/__init__.py`
```python
```
*(empty file)*

---

### File: `app/models/__init__.py`
```python
```
*(empty file)*

---

### File: `app/controller/__init__.py`
```python
```
*(empty file)*

---

### File: `app/infra/__init__.py`
```python
```
*(empty file)*

---

### File: `app/utils/__init__.py`
```python
```
*(empty file)*

---

### File: `app/utils/css_loader.py`
```python
"""Utility to load and inject CSS files into Streamlit apps"""

from pathlib import Path

import streamlit as st
from constants import CSS_DIR


@st.cache_data
def load_css(css_file: Path) -> str:
    """Load CSS content from a file.

    Args:
        css_file: Full path to the CSS file.

    Returns:
        CSS content as a string.
    """
    if not css_file.exists():
        raise FileNotFoundError(f"CSS file not found: {css_file}")

    return css_file.read_text(encoding="utf-8")


@st.cache_data
def inject_css(css_file: Path) -> None:
    """Load and inject CSS into the Streamlit app.

    Args:
        css_file: Full path to the CSS file.
    """
    css_content = load_css(css_file)
    st.markdown(f"<style>{css_content}</style>", unsafe_allow_html=True)
```

---

### File: `app/utils/streamlit_config.py`
```python
"""Streamlit page configuration utilities.

Centralizes page configuration to ensure consistency across all pages
and reduce code duplication.
"""

import streamlit as st


def configure_page(title: str, icon: str = "🏢") -> None:
    """Standard page configuration for all pages.

    Args:
        title: Page title to display in browser tab
        icon: Page icon emoji (defaults to 🏢)

    Note:
        This must be called as the first Streamlit command on each page.
        Streamlit requires set_page_config to be called before any other st command.
    """
    st.set_page_config(
        page_title=title,
        page_icon=icon,
        layout="wide",
        initial_sidebar_state="collapsed",
    )
```

---

### File: `app/static/app.css`
```css
/* Base stylesheet for {{APP_NAME}} */

.center-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 60vh;
    text-align: center;
}

.app-title {
    font-size: 2.5rem;
    font-weight: 500;
    margin-bottom: 0.5rem;
}

.app-subtitle {
    font-size: 1.25rem;
    opacity: 0.7;
}
```

---

### File: `.env.template`
```
# Environment
ENVIRONMENT="local"
```

---

## VERIFICATION (AUTOMATIC)

After writing all files, run these steps automatically:

1. Start Streamlit in headless mode as a background process:
   ```
   uv run streamlit run app/app.py --server.headless true --server.port 8599
   ```
2. Wait 3 seconds for it to start
3. Kill the background process
4. Count how many files were created
5. Report success or any import errors seen in the output

---

## NEXT STEP

After verification, always print this message (substituting actual values):

```
Streamlit app bootstrapped at app/
Files created: {N}

Next steps:
  1. Copy .env.template → .env and fill in values
  2. Run: uv run streamlit run app/app.py
  3. Add your domain models to app/models/
  4. Add your data layer to app/infra/
```
