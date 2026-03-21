---
name: python-project-init
description: Initialize a new Python project with uv following repository conventions. Use when the user asks to "init a python project", "create a new python project", "bootstrap a python project", "initialize a python repo", or "setup a new python package".
disable-model-invocation: true
---

# Python Project Init Skill

Initialize a new Python project with uv, following this repository's conventions.

---

## WORKFLOW

1. Run **DIRECTORY_GUARD**
2. Run **EXISTING_FILE_GUARD**
3. Run **PROCESS** (ask questions, generate files)
4. Run **VERIFICATION (AUTOMATIC)**
5. Print **NEXT STEP** message

---

## DIRECTORY_GUARD

Check the current working directory before doing anything else:

- If a `pyproject.toml` already exists in cwd → use `AskUserQuestion` to ask:
  > "A `pyproject.toml` already exists in this directory. This skill will overwrite it along with other config files.
  >
  > How would you like to proceed?
  > 1. Continue (files will be handled by EXISTING_FILE_GUARD)
  > 2. Abort"
  - If user chooses **2**: stop — print "Aborted. No files were changed."

---

## EXISTING_FILE_GUARD

Check which of these paths already exist:
- `.gitignore`
- `.pre-commit-config.yaml`
- `.python-version`
- `CHANGELOG.md`
- `pyproject.toml`
- `README.md`

If **any** exist, use `AskUserQuestion` to ask:

> "Some files that would be created already exist:
> {list them}
>
> How would you like to proceed?
> 1. Backup existing files (rename to .bak) then overwrite
> 2. Overwrite without backup
> 3. Abort"

- If user chooses **1**: rename each existing file by appending `.bak` suffix, then continue
- If user chooses **2**: continue (overwrite)
- If user chooses **3**: stop — print "Aborted. No files were changed."

---

## PROCESS

Ask the following questions **one at a time** using `AskUserQuestion`. Wait for each answer before asking the next.

### Question 1 — Project name
> "What is the project name? (used in pyproject.toml `name` field, e.g. `my-project`)"

Store as: `PROJECT_NAME`

### Question 2 — Python version
> "Which Python version? (press Enter to use default: `3.11`)"

- If blank → use default `3.11`

Store as: `PYTHON_VERSION`

### Question 3 — Project description
> "What is the project description? (one-line summary for pyproject.toml and README)"

Store as: `PROJECT_DESCRIPTION`

### Question 4 — Author name
> "Author name? (press Enter to skip)"

- If blank → leave author field empty (omit from pyproject.toml)

Store as: `AUTHOR_NAME`

---

After collecting all answers, perform the following steps in order:

### Step 1 — Run uv init

Run:
```
uv init --no-workspace --python {{PYTHON_VERSION}} .
```

This creates a baseline `pyproject.toml`, `README.md`, `.python-version`, `main.py`, and a basic `.gitignore`.

### Step 2 — Remove main.py

Delete `main.py` created by uv init — it is not needed.

### Step 3 — Generate files

Overwrite each file with the full convention-compliant templates below. Replace every placeholder literally:
- `{{PROJECT_NAME}}` → value of PROJECT_NAME
- `{{PYTHON_VERSION}}` → value of PYTHON_VERSION
- `{{PROJECT_DESCRIPTION}}` → value of PROJECT_DESCRIPTION
- `{{AUTHOR_NAME}}` → value of AUTHOR_NAME (only used if not blank)

---

### File: `.gitignore`
```
**/__pycache__/
.ipynb_checkpoints/
*.pkl
.github/

.claude/

data/
!data/.gitkeep

.cache/

.env
.venv/
```

---

### File: `.pre-commit-config.yaml`
```yaml
default_install_hook_types:
  # Mandatory to install both pre-commit and pre-push hooks (see https://pre-commit.com/#top_level-default_install_hook_types)
  # Add new hook types here to ensure automatic installation when running `pre-commit install`
  - pre-commit
  - pre-push
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: check-toml
      - id: check-yaml
      - id: check-json
      - id: check-added-large-files
      - id: check-merge-conflict
      - id: detect-private-key
      - id: trailing-whitespace

  - repo: local
    hooks:
      - id: format-fix
        name: Formatting (ruff)
        entry: uv run ruff format src/ tests/
        pass_filenames: false
        language: system
        types: [python]
        stages: [pre-commit]

      - id: lint-fix
        name: Linting fix (ruff)
        entry: uv run ruff check --fix src/ tests/
        pass_filenames: false
        language: system
        types: [python]
        stages: [pre-commit]

      - id: type-check
        name: Type checking (mypy)
        entry: >
           uv run mypy
            --explicit-package-bases
            src/ tests/
        pass_filenames: false
        language: system
        types: [python]
        stages: [pre-commit]

      - id: cyclomatic-complexity
        name: Cyclomatic Complexity check (radon/xenon)
        entry: >
          uv run xenon src/ tests/
            --max-average B
            --max-module B
            --max-absolute C
        pass_filenames: false
        language: system
        types: [python]
        stages: [pre-commit]
```

---

### File: `.python-version`
```
{{PYTHON_VERSION}}
```

---

### File: `CHANGELOG.md`
```
```
*(empty file)*

---

### File: `pyproject.toml`

If `AUTHOR_NAME` is **not blank**, use this template:
```toml
[project]
name = "{{PROJECT_NAME}}"
version = "0.1.0"
description = "{{PROJECT_DESCRIPTION}}"
readme = "README.md"
requires-python = ">={{PYTHON_VERSION}}"
authors = [{ name = "{{AUTHOR_NAME}}" }]
dependencies = []

[dependency-groups]
dev = [
    "mypy>=1.0.0",
    "pre-commit>=4.0.0",
    "pytest>=8.0.0",
    "radon[toml]>=6.0.1",
    "ruff>=0.1.0",
    "xenon>=0.9.3",
]

[tool.ruff]
line-length = 120

[tool.mypy]
python_version = "{{PYTHON_VERSION}}"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
explicit_package_bases = true

[tool.pytest.ini_options]
testpaths = ["tests"]
```

If `AUTHOR_NAME` **is blank**, omit the `authors` line:
```toml
[project]
name = "{{PROJECT_NAME}}"
version = "0.1.0"
description = "{{PROJECT_DESCRIPTION}}"
readme = "README.md"
requires-python = ">={{PYTHON_VERSION}}"
dependencies = []

[dependency-groups]
dev = [
    "mypy>=1.0.0",
    "pre-commit>=4.0.0",
    "pytest>=8.0.0",
    "radon[toml]>=6.0.1",
    "ruff>=0.1.0",
    "xenon>=0.9.3",
]

[tool.ruff]
line-length = 120

[tool.mypy]
python_version = "{{PYTHON_VERSION}}"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
explicit_package_bases = true

[tool.pytest.ini_options]
testpaths = ["tests"]
```

---

### File: `README.md`
```markdown
# {{PROJECT_NAME}}

{{PROJECT_DESCRIPTION}}

## Prerequisites

- Python {{PYTHON_VERSION}}+
- [uv](https://docs.astral.sh/uv/) package manager

## Installation

Install uv if not already installed:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Install dependencies:
```bash
# Base dependencies only
uv sync

# With dev dependencies
uv sync --group dev
```

## Managing Dependencies

```bash
# Add a dependency
uv add <package>

# Add a dev dependency
uv add --dev <package>

# Remove a dependency
uv remove <package>
```

## Code Quality

Install pre-commit hooks:
```bash
uv run pre-commit install
```

Run all checks manually:
```bash
uv run pre-commit run --all-files
```

## Project Structure

```
{{PROJECT_NAME}}/
├── src/                  # Source code
├── tests/                # Test suite
├── .pre-commit-config.yaml
├── .python-version
├── pyproject.toml
└── README.md
```

## Development

TODO: describe how to run and develop this project.
```

---

## VERIFICATION (AUTOMATIC)

After writing all files, run these steps automatically:

1. Run:
   ```
   uv sync --group dev
   ```
2. Count how many files were created/overwritten (should be 6: `.gitignore`, `.pre-commit-config.yaml`, `.python-version`, `CHANGELOG.md`, `pyproject.toml`, `README.md`)
3. Report success or any errors seen in the output

---

## NEXT STEP

After verification, always print this message (substituting actual values):

```
Python project initialized at ./
Files created: 6

  .gitignore
  .pre-commit-config.yaml
  .python-version
  CHANGELOG.md
  pyproject.toml
  README.md

Next steps:
  1. Install pre-commit hooks:   uv run pre-commit install
  2. Add your source code to     src/
  3. Add your tests to           tests/
  4. Track changes in            CHANGELOG.md
```
