# uv Cheatsheet

uv is a fast Python package and project manager. It can manage Python versions, virtual environments, dependencies, lockfiles, tools, and project workflows.

---

## Installing uv

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# With Homebrew (macOS)
brew install uv

# Verify installation
uv --version
```

> After installation, restart your shell or reload your shell configuration if necessary.

---

## Initialising a Project

```bash
# Initialise a new uv project in the current directory
uv init

# Initialise with a specific project name
uv init my-project

# Initialise an application project
uv init --app my-project

# Initialise a library project
uv init --lib my-project

# Specify the Python version
uv init --python 3.11
```

### What gets created

```text
my-project/
├── pyproject.toml    # Project configuration and dependencies
├── README.md
├── main.py           # Entry point (application projects)
└── .python-version   # Python version used by the project
```

`uv init --lib` creates a `src/` layout instead, with the package code under `src/<project_name>/`.

---

## Common uv Commands for Managing Projects

```bash
# Create/update the project environment and install dependencies
uv sync

# Run a command inside the project environment
uv run python main.py

# Run a Python module
uv run python -m myapp

# Run tests
uv run pytest

# Show project dependency tree
uv tree

# Lock dependencies without installing
uv lock

# Update dependencies
uv lock --upgrade
```

---

## Managing Dependencies

```bash
##############################################################
# Adding packages

# Add a package
uv add numpy

# Add a package with a version constraint
uv add "numpy>=1.24"

# Add a specific version
uv add "pandas==2.2.3"

# Add multiple packages
uv add numpy pandas scikit-learn

# Add a development dependency
uv add --dev pytest

# Add multiple development dependencies
uv add --dev ruff black pytest

##############################################################
# Removing packages

# Remove a package
uv remove numpy

# Remove a development dependency
uv remove --dev pytest
```

---

## The `pyproject.toml` Manifest

The `pyproject.toml` file defines project metadata, Python requirements, dependencies, optional dependencies, and development configuration.

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "My uv project"
requires-python = ">=3.11"

dependencies = [
    "numpy>=1.24",
    "pandas>=2.0",
    "scikit-learn",
]

[dependency-groups]
dev = [
    "pytest",
    "ruff",
    "black",
]
```

---

## Managing Python Versions

uv can install and manage Python versions independently of the system Python installation.

```bash
# List available Python versions
uv python list

# Install a Python version
uv python install 3.11

# Install another version
uv python install 3.12

# Find the path to a Python installation
uv python find 3.11

# Pin the project to a Python version
uv python pin 3.11

# Show installed Python versions
uv python list --only-installed
```

### `.python-version`

```text
3.11
```

The `.python-version` file tells uv which Python version the project should use.

---

## Virtual Environments

uv can create and manage standard Python virtual environments.

```bash
# Create a virtual environment
uv venv

# Create one with a specific Python version
uv venv --python 3.11

# Create a named environment
uv venv .venv

# Activate on macOS / Linux
source .venv/bin/activate

# Activate on Windows PowerShell
.venv\Scripts\Activate.ps1

# Deactivate
deactivate
```

> For normal uv projects, `uv sync` automatically creates and manages the project's `.venv`, so manually running `uv venv` is often unnecessary.

---

## Running Commands

One of uv's main advantages is that commands can be executed without manually activating the environment.

```bash
# Run Python
uv run python

# Run a script
uv run python script.py

# Run a module
uv run python -m myapp

# Run pytest
uv run pytest

# Run Ruff
uv run ruff check .

# Run Jupyter
uv run jupyter lab
```

This is particularly useful for scripts and CI because the correct environment is selected automatically.

---

## Managing Tasks

uv does not have a built-in task runner equivalent to Pixi tasks. Common commands can instead be defined as scripts using project configuration or run directly through `uv run`.

```bash
# Run a development server
uv run python -m myapp

# Run tests
uv run pytest tests/

# Run linting
uv run ruff check .

# Format code
uv run ruff format .
```

For larger projects, task runners such as `just`, `make`, or `nox` can be combined with uv.

---

## Dependency Groups

uv supports dependency groups for separating development, testing, documentation, and other optional dependencies.

```toml
[dependency-groups]
dev = [
    "ruff",
    "black",
]

test = [
    "pytest",
    "pytest-cov",
]

docs = [
    "mkdocs",
    "mkdocs-material",
]
```

```bash
# Install the default dependency groups
uv sync

# Install a specific group
uv sync --group test

# Install multiple groups
uv sync --group test --group docs

# Exclude a group
uv sync --no-group docs
```

---

## Lockfile

uv automatically maintains a lockfile:

```text
uv.lock
```

```bash
# Generate/update the lockfile
uv lock

# Upgrade all dependencies
uv lock --upgrade

# Upgrade a specific package
uv lock --upgrade-package numpy

# Install exactly according to the lockfile
uv sync --locked
```

> **Always commit `uv.lock`** to version control for reproducible environments.

---

## Using uv with VS Code

VS Code can use the `.venv` created by uv.

1. Open the Command Palette (`Ctrl + Shift + P` or `Cmd + Shift + P`).
2. Select **Python: Select Interpreter**.
3. Select the project's `.venv` interpreter.
4. If it is not listed, choose **Enter interpreter path...**.

Typical locations:

```text
# Windows
.venv\Scripts\python.exe

# macOS / Linux
.venv/bin/python
```

For Jupyter:

1. Open the notebook.
2. Click **Select Kernel**.
3. Select the project's `.venv` Python environment.

---

## Using uv with Jupyter

```bash
# Add Jupyter
uv add --dev jupyterlab ipykernel

# Launch JupyterLab
uv run jupyter lab

# Register the environment as a Jupyter kernel
uv run python -m ipykernel install --user \
    --name my-project \
    --display-name "Python (my-project)"
```

---

## Installing Global CLI Tools

uv can install Python-based command-line tools independently of project environments.

```bash
# Install a tool
uv tool install ruff

# Install another tool
uv tool install black

# Install IPython
uv tool install ipython

# List installed tools
uv tool list

# Upgrade a tool
uv tool upgrade ruff

# Upgrade all tools
uv tool upgrade --all

# Remove a tool
uv tool uninstall ruff

# Run a tool without permanently installing it
uvx ruff check .
```

`uvx` is especially useful for one-off CLI tools.

---

## Running Tools with `uvx`

```bash
# Run Ruff without installing it permanently
uvx ruff check .

# Run Black
uvx black .

# Run a specific version
uvx ruff@0.8.0 check .

# Run a tool with additional arguments
uvx pytest tests/
```

> `uvx` is roughly analogous to running a temporary isolated tool environment.

---

## Migrating to uv

### From `requirements.txt`

Given:

```text
numpy>=1.24
pandas>=2.0
scikit-learn
requests
```

Migration:

```bash
# 1. Create a uv project
uv init my-project

cd my-project

# 2. Add the requirements
uv add numpy "pandas>=2.0" scikit-learn requests

# 3. Synchronise the environment
uv sync
```

> For an existing project, dependencies can also be migrated into `pyproject.toml` manually and then locked with `uv lock`.

---

### From `venv` + `pip`

```bash
# Existing requirements.txt
uv venv

# Install requirements into the uv-managed environment
uv pip install -r requirements.txt

# Or migrate the project to uv's project workflow
uv init
uv add <packages>
uv sync
```

`uv pip` provides a pip-compatible interface for environments where a full migration to the project workflow is not yet appropriate.

---

### From Conda

For projects currently using `environment.yml`:

```bash
# Create a uv project
uv init my-project

cd my-project

# Install Python
uv python install 3.11

# Add Python packages
uv add numpy pandas scikit-learn
```

> uv is primarily a Python package/project manager and does not replace Conda's broader system-package and non-Python dependency ecosystem. Packages that require Conda-specific binaries may need a different approach.

---

## Example: Setting Up a Data Science Project from Scratch

```bash
# 1. Create the project
uv init data-science-project

cd data-science-project

# 2. Pin Python
uv python install 3.11
uv python pin 3.11

# 3. Add core data science packages
uv add numpy pandas scikit-learn matplotlib seaborn

# 4. Add Jupyter
uv add --dev jupyterlab ipykernel

# 5. Add development tools
uv add --dev pytest ruff

# 6. Create/update the environment
uv sync

# 7. Launch JupyterLab
uv run jupyter lab

# 8. Run tests
uv run pytest
```

---

## Useful Project Structure

```text
data-science-project/
├── pyproject.toml
├── uv.lock
├── .python-version
├── .venv/
├── src/
│   └── my_project/
├── tests/
├── notebooks/
└── README.md
```

Add `.venv/` to `.gitignore`:

```text
.venv/
```

---

## Key Differences: pip / venv vs uv

| Feature | pip + venv | uv |
|---|---|---|
| Package manager | `pip` | `uv` |
| Environment | `python -m venv` | `uv venv` / automatic |
| Manifest | `requirements.txt` | `pyproject.toml` |
| Lockfile | Not built-in | `uv.lock` |
| Install dependencies | `pip install` | `uv add` / `uv sync` |
| Run without activation | ✗ | `uv run COMMAND` |
| Python management | System / pyenv | Built-in `uv python` |
| Global tools | `pipx` | `uv tool` |
| Temporary tools | — | `uvx` |
| Speed | Moderate | Very fast (Rust) |
| Project management | Separate tools | Built-in |

---

## Key Differences: Conda vs uv

| Feature | Conda | uv |
|---|---|---|
| Primary ecosystem | Python + native packages | Python |
| Manifest file | `environment.yml` | `pyproject.toml` |
| Lockfile | `conda-lock` / environment spec | `uv.lock` |
| Environment | Conda environment | `.venv` |
| Activation | `conda activate` | Usually unnecessary |
| Run without activation | Limited | `uv run COMMAND` |
| PyPI support | Via pip | Native |
| Python management | Yes | Yes |
| Non-Python packages | Strong | Limited |
| Global CLI tools | Conda environments | `uv tool` |
| Speed | Moderate | Very fast |

---

## Tips & Best Practices

- **Commit `uv.lock`** — it provides reproducible dependency resolution across machines and CI.
- **Use `uv add` rather than manually editing dependencies** when possible.
- **Use `uv sync`** to keep the environment aligned with `pyproject.toml` and `uv.lock`.
- **Use `uv run`** in scripts and CI so commands run in the correct environment without activation.
- **Use `uv add --dev`** for development-only packages such as pytest and Ruff.
- **Use `uv tool install`** for globally available Python CLI applications.
- **Use `uvx`** for one-off tools that do not need to remain installed.
- **Pin Python with `.python-version`** when the project requires a specific Python version.
- **Do not commit `.venv/`** — the environment should be recreated from the project metadata and lockfile.
- **Prefer uv's project workflow** (`pyproject.toml` + `uv.lock` + `uv sync`) for new Python projects.
- **Use `uv pip`** when you specifically need pip-compatible environment management.

```text
# .gitignore

.venv/
```