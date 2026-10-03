# Ruff Quick Guide

> Rule defaults change between releases, so check the [Ruff docs](docs.astral.sh/ruff) for your installed version.

## Linter and Formatter

A **linter** reads your code *without running it* and flags problems, including:
- **Bugs:** unused variables, undefined names, `==` against `None`
- **Style:** line length, import order, naming conventions
- **Modernization:** old syntax that newer Python versions replace
- **Security and complexity:** hardcoded passwords, overly branchy functions

A **formatter** rewrites layout (formatting styles, e.g., spacing, double vs single quotes, wrapping) and never changes behavior. Linters tell you what's wrong, while formatters just fix the layout. **Ruff** does both.

## Get started

```bash
pip install ruff          # or: uv add --dev ruff
ruff check .              # lint
ruff check --fix .        # lint + auto-fix safe issues
ruff format .             # format
```

Try it on this file:

```python
import os, sys

def f(x):
    unused = 1
    if x == None:
        return "none"
```

`ruff check` will report an unused import (`F401`), multiple imports on one line (`E401`), an unused variable (`F841`), and a comparison to `None` (`E711`).

## Rules and selection

Rules:
- Every rule has a code: a **letter prefix** (the rule family) plus a number. 
- By default, Ruff:
  - enables only a small set (`E4`, `E7`, `E9`, `F`). You opt into more families.
  - uses a line length of 88. 

| Prefix | Family | What it catches |
|---|---|---|
| `F` | Pyflakes | Unused imports/variables, undefined names |
| `E` / `W` | pycodestyle | Style errors and warnings |
| `I` | isort | Import sorting |
| `B` | bugbear | Likely bugs and gotchas |
| `UP` | pyupgrade | Outdated syntax |
| `N` | pep8-naming | Naming conventions |
| `SIM` | simplify | Needlessly complex code |
| `C4` | comprehensions | Better list/dict/set comprehensions |
| `S` | bandit | Security issues |
| `PL` | Pylint | Assorted Pylint checks |
| `RUF` | Ruff-specific | Ruff's own rules |

Explore from the terminal: 
- `ruff rule F401` explains a rule
- `ruff linter` lists all families

## Configuration

Put config in `pyproject.toml` (or in `ruff.toml`, without the `tool.ruff` prefix):

```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM"]
ignore = ["E501"]           # line length handled by the formatter

[tool.ruff.lint.per-file-ignores]
"tests/*" = ["S101"]        # allow assert in tests
"__init__.py" = ["F401"]    # allow re-exports

[tool.ruff.format]
quote-style = "double"
```

A good workflow is to start with the defaults and add one family at a time.

## Suppressing and fixing

**Fixes** can be:
- safe: behavior-preserving, applied by `--fix`
- unsafe: might change behavior, only applied with `--unsafe-fixes`. Review the diff before committing unsafe ones.

Ruff rules can be suppressed at three levels of scope: single lines, contiguous code blocks, or entire files.
- **Single line:** Append `# noqa: <RULE>` (e.g., `x = 1  # noqa: F841`). Avoid bare `# noqa`, which hides all violations indiscriminately.
- **Code block:** Wrap a contiguous section using `# fmt: skip` or boundary comments.
* **Entire file:** Add `# ruff: noqa: <RULE>` (e.g., `# ruff: noqa: F401`) near the top of the file.
* **Best practice:** Prefer per-file ignores in configuration files over scattered inline `# noqa` comments.
* **Legacy adoption:** Run `ruff check --add-noqa` to silence all existing violations and address them incrementally over time.

## Integrate Ruff across your local setup, CI pipeline, and editor


**Pre-commit (`.pre-commit-config.yaml`):** Automatically fix lint issues and format staged code before commits.

  ```yaml
  repos:
    - repo: https://github.com/astral-sh/ruff-pre-commit
      rev: v0.x.x          # Pin to a specific release
      hooks:
        - id: ruff-check
          args: [--fix]
        - id: ruff-format
  ```

**CI Pipeline:** Run non-modifying verification checks that exit with a non-zero status code on errors.
  
```bash
ruff check .
ruff format --check .

```

**Editor Support:** Install the official Ruff extension (VS Code, Neovim, PyCharm, etc.) to enable real-time linting and format-on-save.



---

# Ruff cheatsheet

**Commands**

| Task | Command |
|---|---|
| Lint | `ruff check .` |
| Lint and fix (safe) | `ruff check --fix .` |
| Include unsafe fixes | `ruff check --fix --unsafe-fixes .` |
| Watch mode | `ruff check --watch .` |
| Format | `ruff format .` |
| Check formatting only | `ruff format --check .` |
| Preview format changes | `ruff format --diff .` |
| Count violations by rule | `ruff check --statistics .` |
| Explain a rule | `ruff rule E711` |
| List rule families | `ruff linter` |
| Auto-add `noqa` to everything | `ruff check --add-noqa .` |
| One-off run without installing | `uvx ruff check .` |

**Useful flags:** `--select B,I` (override rules), `--ignore E501`, `--output-format=concise`, `--diff` (show fixes without applying).

**Suppression syntax**

```python
x = 1                    # noqa: F841 (one line, one rule)
import os                # noqa: F401, E402 (multiple rules)
# ruff: noqa: F401 (entire file)
```

**Config skeleton**

```toml
[tool.ruff]
line-length = 88
[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP"]
ignore = []
[tool.ruff.lint.per-file-ignores]
"tests/*" = ["S101"]
[tool.ruff.format]
quote-style = "double"
```

**Recommended starter set:** `E`, `F`, `I`, `B`, `UP`, `SIM`, `RUF`. Add `S` for security and `PL` for deeper checks once the basics are clean.

**Good Habits**

1. Run `ruff check --fix` and `ruff format` before every commit (via pre-commit).
2. Run the non-modifying versions in CI pipelines.
3. Adopt rule families one at a time.
4. Always use specific `noqa` codes.
5. Pin the Ruff version in your project, since new releases can add rules and change formatting slightly.