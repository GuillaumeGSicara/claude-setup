# Setting Management

## Overview

All configuration for this project must be loaded through **pydantic-settings**.
The single source of truth is the `ProjectSettings` class.

Values are read from a **`.env`** (generated from the provided `.env.template`) file in the project root OR from real environment variables (e.g injected at pod bootstrap for a runtime) - **never** by calling `os.getenv` or by hardcoding values in code.

## ✅ Correct Conventions

### 1️⃣ Declaring a new setting
*Add the field to a settings model and a placeholder in the `.env.template` file.*

```python
# settings.py
class ProjectSettings(BaseSettings):
    # existing settings...
    new_api_endpoint: str = Field(..., description="The URL of the new API endpoint")
    new_api_key: SecretStr = Field(..., description="The API key for authenticating with the new API")
```

```dotenv
# .env.template
# existing settings...
NEW_API_ENDPOINT="https://api.example.com/v1"
NEW_API_KEY="XXXXXXX"
```

* Use `SecretStr` for any sensitive values (e.g. API keys, database passwords)
* Do **not** provide defaults in the model `Field()`; let pydantic-settings handle missing values and validation


### 2️⃣ Creating a new settings class
* Subclass an existing settings model when a feature needs its own group of options.*

```python
# streamlit_app/settings.py
class FeatureSettings(ProjectSettings):
   """Feature specific settings for the Feature app."""
   model_config = SettingsConfigDict(
      env_file=PROJECT_ROOT / ".env", env_file_encoding="utf-8", extra="ignore"
   )
   # Example field
    feature_secret: SecretStr = Field(..., description="The secret for the Feature app")
```

* Inherit from `ProjectSettings`
* Keep the same `model_config` so the single `.env` remains the source of truth for all settings


### 3️⃣ Accessing settings throughout the codebase


| Context | Implementation Example | Recommended pattern |
| --- | --- | --- |
| **Application start** | `settings = ProjectSettings() # type: ignore` in `src/main.py` | Create **one** instance and pass it onwards |
| ** Functions/infra ** | `def my_function(settings: ProjectSettings): ...` | Pass the settings instance as an argument |
| **Secrets** | `settings.api_key.get_secret_value()` | Use `get_secret_value()` to access the raw value of `SecretStr` fields |
| **Streamlit UI** | `state: StateHandler = StateHandler(); state.settings.execution_env` | Pages instantiate `StateHandler` once and read/write


**Quick rule of thumb**
- **Inject** the existing settings objects [`settings` or `state.settings`] into any function that needs configuration.
- **Never** call `os.getenv` or create a new settings instance inside helpers.
- **Never** hard-code config value or secrets


Following these three steps-declare the field, optionally subclass for a feature, and always inject the same settings instance-keeps configuration typed, centralised, and secure across the codebase.


## 🚫 What **must not** be done (Anti-patterns)

```python
# BAD - reading env vars directly in code
api_key = os.getenv("API_KEY")

# BAD - creating a new settings instance in a function
def fetch_data():
    settings = ProjectSettings() # type: ignore
    api_key = settings.api_key.get_secret_value()
    ...

# BAD - accessing settings via streamlit session state
token = st.session_state.get("token")  # Avoid this! Use a StateHandler instead and read from settings.

# Bad - Hard-coding a secret value
API_KEY = "abdc1312452"  # keep secrets in the .env + SecretStr 

# BAD - mutating os.environ directly
os.environ["API_URL"] = "https://api.example.com"  # Don't do this! Use a settings model and .env file instead.

# BAD - mixing config access with business logic
def process():
    if settings.llmaas_url.get_secret_value().startswith("https"):
        # business logic mixed with config access - avoid this!
        ...
```

## 📦 How to Apply the Rule

1. **Create/Update Settings**
  * Add any new configuration item to `ProjectSettings` or a feature-specific subclass.
  * Add a matching entry to `.env.template` with a placeholder value.
2. **Use Dependency Injection**
  * Pass the settings instance (`settings` or `state.settings`) to any function or component that needs configuration.
  * In Streamlit pages, read the settings via `state.settings` after instantiating `StateHandler`.
4. **Never Reach Directly into `os.getenv`**
 * If you find a `os.getenv` call, refactor to move that config item into the settings model and read it from there instead. 
4. **Secret Handling**
  * For any sensitive values, use `SecretStr` in the settings model and access the raw value with `get_secret_value()`.
  * Never hard-code secrets or read them directly from environment variables in code.
