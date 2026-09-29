# Pixi Cheatsheet

Pixi is a fast, cross-platform package manager built on top of the conda ecosystem. It uses `pixi.toml` (or `pyproject.toml`) to manage project dependencies reproducibly.

---

## Installing Pixi

```bash
# macOS / Linux
curl -fsSL https://pixi.sh/install.sh | bash

# Windows (PowerShell)
iwr -useb https://pixi.sh/install.ps1 | iex

# With Homebrew (macOS)
brew install pixi

# Verify installation
pixi --version
```

> After installation, restart your shell or run `source ~/.bashrc` / `source ~/.zshrc` to reload your PATH.

---

## Initialising a Project

```bash
# Initialise a new Pixi project in the current directory (creates pixi.toml)
pixi init

# Initialise with a specific name
pixi init my-project

# Initialise and use pyproject.toml instead of pixi.toml
pixi init --format pyproject

# Initialise with a specific channel
pixi init --channel conda-forge --channel defaults
```

### What gets created

```
my-project/
├── pixi.toml       # Project manifest (dependencies, tasks, environments)
└── .pixi/          # Local environment cache (add to .gitignore)
```

---

## Common Pixi Commands for Managing Projects

```bash
# Install all dependencies defined in pixi.toml
pixi install

# Run a command inside the Pixi environment (without activating it)
pixi run python script.py

# Activate the Pixi environment in your current shell
pixi shell

# Exit the activated Pixi shell
exit

# Show information about the current project and environment
pixi info

# List all installed packages
pixi list

# Update all packages to their latest compatible versions
pixi update

# Update a specific package
pixi update numpy
```

---

## Managing Dependencies

```bash
##############################################################
# Adding packages

# Add a package (from conda-forge by default)
pixi add numpy

# Add a package with a version constraint
pixi add "numpy>=1.24"

# Add a package from a specific channel
pixi add --channel bioconda samtools

# Add a PyPI package
pixi add --pypi requests

# Add a PyPI package with a version constraint
pixi add --pypi "flask>=3.0"

# Add a package only to a specific feature/environment
pixi add numpy --feature data-science

##############################################################
# Removing packages

# Remove a package
pixi remove numpy

# Remove a PyPI package
pixi remove --pypi requests

##############################################################
# Adding channels globally

# Add a channel to the project
pixi project channel add bioconda

# Remove a channel from the project
pixi project channel remove bioconda

# List configured channels
pixi project channel list
```

---

## The `pixi.toml` Manifest

The `pixi.toml` file is the heart of a Pixi project. It defines dependencies, channels, tasks, and multiple environments.

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "My Pixi project"
channels = ["conda-forge", "defaults"]
platforms = ["linux-64", "osx-arm64", "win-64"]

[dependencies]
python = ">=3.11"
numpy = ">=1.24"
pandas = ">=2.0"
scikit-learn = "*"

[pypi-dependencies]
some-niche-pip-package = "*"
another-library = "==1.2.3"

[tasks]
start   = "python main.py"
test    = "pytest tests/"
lint    = "ruff check ."
format  = "black ."

[feature.dev.dependencies]
pytest = "*"
ruff   = "*"
black  = "*"

[environments]
default = { features = ["dev"], solve-group = "default" }
prod    = { features = [], solve-group = "default" }
```

---

## Managing Tasks

Pixi tasks replace `Makefile` targets or shell scripts for common project commands.

```bash
# Run a task defined in pixi.toml
pixi run start
pixi run test
pixi run lint

# List all available tasks
pixi task list

# Add a task from the CLI
pixi task add train "python train.py --epochs 10"

# Remove a task
pixi task remove train

# Run a task in a specific environment
pixi run --environment prod start
```

---

## Managing Multiple Environments

Pixi supports multiple named environments in a single project (e.g. `default`, `prod`, `docs`).

```bash
# Install dependencies for all environments
pixi install --all

# Install a specific environment
pixi install --environment prod

# Activate a specific environment
pixi shell --environment prod

# Run a command in a specific environment
pixi run --environment prod python main.py

# List all environments
pixi project environment list
```

### Multi-environment `pixi.toml` example

```toml
[feature.test.dependencies]
pytest       = "*"
pytest-cov   = "*"

[feature.docs.dependencies]
mkdocs            = "*"
mkdocs-material   = "*"

[environments]
default = { features = ["test"] }
docs    = { features = ["docs"], no-default-feature = true }
```

---

## Using Pixi with VS Code

If VS Code doesn't detect the project's Pixi environment automatically, you need to connect it to VS Code manually:

1. Open the Command Palette (`Ctrl + Shift + P` or `Cmd + Shift + P`).
2. Select "Python: Select Interpreter".
3. Choose "Enter interpreter path..." → "Find...."
4. Navigate to your project folder and select the Python executable:
  - Windows: `.pixi/envs/default/python.exe`
  - macOS / Linux: `.pixi/envs/default/bin/python`
5. For Jupyter notebook: click "Select Kernel" (top right) → "Python Environments...", and select the newly added Pixi kernel.

## Using Pixi with Jupyter

```bash
# Add JupyterLab to the project
pixi add jupyterlab ipykernel

# Launch JupyterLab inside the Pixi environment
pixi run jupyter lab

# Register the environment as a Jupyter kernel (optional)
pixi run python -m ipykernel install --user --name my-project
```

---

## Global Tool Installation

Pixi can install CLI tools globally (similar to `pipx` or `conda install -n base`).

```bash
# Install a tool globally
pixi global install ruff
pixi global install black
pixi global install ipython

# List globally installed tools
pixi global list

# Remove a globally installed tool
pixi global remove ruff

# Upgrade a globally installed tool
pixi global upgrade ruff

# Upgrade all global tools
pixi global upgrade-all
```

---

## Migrating to Pixi

### From Conda (`environment.yml`)

Given an existing `environment.yml`:

```yaml
name: myenv
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.11
  - numpy
  - pandas
  - pip:
    - flask>=3.0
    - some-niche-package
```

**Migration steps:**

```bash
# 1. Initialise a new Pixi project
pixi init my-project
cd my-project

# 2. Add channels to match your environment.yml
pixi project channel add conda-forge

# 3. Add conda dependencies
pixi add "python=3.11" numpy pandas

# 4. Add pip/PyPI dependencies
pixi add --pypi "flask>=3.0" some-niche-package

# 5. (Optional) Verify all packages are installed correctly
pixi install
pixi list
```

> **Tip:** Pixi can also import an `environment.yml` directly:
> ```bash
> pixi init --import environment.yml
> ```

---

### From `requirements.txt` (pip / venv)

```bash
# 1. Initialise a Pixi project
pixi init my-project
cd my-project

# 2. Add Python
pixi add python=3.11

# 3. Read requirements.txt and add each package as a PyPI dependency
pixi add --pypi $(cat requirements.txt | tr '\n' ' ')

# — OR — add them one by one for more control
pixi add --pypi flask requests pandas
```

> For complex `requirements.txt` files with markers or extras, add packages manually and verify with `pixi list`.

---

### From Poetry (`pyproject.toml`)

```bash
# 1. Initialise Pixi using the existing pyproject.toml
pixi init --format pyproject

# 2. Pixi will add its own [tool.pixi] section to pyproject.toml
#    Add your channels
pixi project channel add conda-forge

# 3. Migrate dependencies from [tool.poetry.dependencies] to Pixi
#    Add conda packages
pixi add numpy pandas scikit-learn

# 4. Add remaining PyPI-only packages
pixi add --pypi some-pypi-only-package

# 5. Migrate scripts / entry points to Pixi tasks
pixi task add start "python -m myapp"
pixi task add test  "pytest"
```

---

### From `conda-lock` / `pip-tools`

```bash
# Pixi generates its own lockfile: pixi.lock
# It is created automatically on `pixi install` — commit it to version control!

# To regenerate the lockfile from scratch
pixi update

# To reproduce an environment exactly from the lockfile (CI / teammates)
pixi install   # reads pixi.lock if present
```

---

## Example: Setting Up a Data Science Project from Scratch

```bash
# 1. Create and enter the project
pixi init data-science-project
cd data-science-project

# 2. Add core data science packages (conda)
pixi add python=3.11 numpy pandas scikit-learn matplotlib seaborn jupyterlab ipykernel

# 3. Add any PyPI-only packages
pixi add --pypi shap plotly-express

# 4. Add dev tools as a feature
pixi add --feature dev pytest ruff black

# 5. Define handy tasks
pixi task add notebook "jupyter lab"
pixi task add test     "pytest tests/"
pixi task add lint     "ruff check . && black --check ."

# 6. Launch JupyterLab
pixi run notebook
```

---

## Key Differences: Conda vs Pixi

| Feature | Conda | Pixi |
|---|---|---|
| Manifest file | `environment.yml` | `pixi.toml` / `pyproject.toml` |
| Lockfile | `conda-lock.yml` (optional) | `pixi.lock` (automatic) |
| Activation | `conda activate MYENV` | `pixi shell` |
| Run without activating | ✗ | `pixi run COMMAND` |
| PyPI support | Via pip (manual) | Native `--pypi` flag |
| Multiple environments | Multiple `.yml` files | Built-in features & environments |
| Task runner | ✗ (use Makefile) | Built-in `pixi task` |
| Global tool install | `conda install -n base` | `pixi global install` |
| Speed | Moderate | Fast (built in Rust) |

---

## Tips & Best Practices

- **Always commit `pixi.lock`** — it guarantees reproducible environments for your whole team and CI pipelines.
- **Prefer conda packages over PyPI** — use `--pypi` only for packages not available on conda channels.
- **Use features for optional dependency groups** — keep dev, test, and docs dependencies separate from production dependencies.
- **Avoid mixing with system conda** — do not run `conda activate` inside a `pixi shell`; they can conflict.
- **Add `.pixi/` to `.gitignore`** — the local environment cache should never be committed.
- **Use `pixi run` in scripts and CI** — it works without activating the environment, making it ideal for automation.

```bash
# .gitignore entry
.pixi/
```
