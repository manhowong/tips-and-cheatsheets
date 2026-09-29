# AI-Assisted Development: Agent Customization

This guide covers customization for three tools: GitHub Copilot Chat and Cline (VS Code extensions), and Aider (a CLI tool). For basic setup and usage, see `AI-assisted-dev-quickstart.md`.

## Common shortcuts and context referencing

### Default shortcuts

**GitHub Copilot**

| Action | Windows / Linux | macOS | Purpose |
| ---------------------------- | ------------------ | ------ | -------------------------------------------- |
| Toggle Chat view             | `Ctrl+Alt+I`       | `Ctrl+Cmd+I`  | Open the Chat view |
| Inline chat                  | `Ctrl+I`           | `Cmd+I`   | Chat in the current editor or terminal |
| Quick Chat                   | `Ctrl+Shift+Alt+L` | `Cmd+Shift+Opt+L` | Ask a short question without opening Chat view |
| Accept inline suggestion     | `Tab`              | `Tab`  | Accept an inline completion |
| Dismiss inline suggestion    | `Esc`              | `Esc`  | Reject an inline completion |

To start an agent-oriented chat, open the Chat view and select Agent in the mode picker. Check Keyboard Shortcuts in your VS Code version for any dedicated shortcut.

**Cline**

| Action | Windows / Linux | macOS | Purpose |
| --- | --- | --- | --- |
| Open Chat / Add highlighted content  | `Ctrl+'`            | `Cmd+'`       | Opens input or sends highlighted code to Cline. |
| *Toggle Plan / Act Mode | `Win/Super+Shift+A` | `Cmd+Shift+A` | Switches between planning and execution modes. |

*Only works inside Cline sidbar panel.

### Context referencing

**GitHub Copilot**

| Context | Syntax / mechanism |
|---|---|
| Active editor tab / selection | Automatically included                  |
| File reference                | `#file` + filename                      |
| Folder reference              | `#` + folder name                       |
| Symbol                        | `#` + symbol name                       |
| Codebase (workspace)          | `#codebase`                             | 
| Terminal output               | `#terminalSelection` / terminal context |
| Tool                          | `#` + tool name (e.g., `#fetch`)        |

VS Code also supports `@` mentions for chat participants such as `@vscode` and `@terminal`.

**Cline**

Here is the matching structured guide for Cline:
Cline

| Context | Syntax / mechanism |
|---|---|
| Active editor tab | Automatically included                                 |
| Current selection | `Ctrl/Cmd+'` or Editor -> Right-Click -> Add to Cline  |
| File reference    | `@` + filename                                         |
| Folder reference  | `@` + folder name                                      |
| Terminal output   | `@terminal` or Terminal -> Right-Click -> Add to Cline |
| Workspace errors  | `@problems`                                            |
| Web content       | Paste URL                                              |

**Aider**

- `/add <file>` adds a file into the chat's context.
- `/ls` lists files currently in context.
- `/map` shows Aider's repository map, a compressed outline of the codebase it uses for context without loading every file.

## Agent customization

### Persistent rules

**GitHub Copilot**

Create `AGENTS.md` at the repo root, or run `/init` in Chat to generate `.github/copilot-instructions.md` from your codebase. Both are read automatically on every request.

**Cline**

Create a `.clinerules/` folder with one or more `.md` files. Everything in the folder is read automatically.

**Aider**

Create `CONVENTIONS.md`. Aider does not read it automatically; add `read: CONVENTIONS.md` to `.aider.conf.yml`, or pass `--read CONVENTIONS.md` on each run.

### Path-specific rules

**GitHub Copilot**

Create files under `.github/instructions/` ending in `.instructions.md`, scoped with `applyTo`:

```markdown
---
applyTo: "**/*.py"
---

# Python Instructions

- Use Ruff.
- Follow the project's Python version.
- Use the project's established testing conventions.
```

**Cline**

No native scoping. Workaround: add a separate file per location inside `.clinerules/`, with the scope stated in the file itself.

**Aider**

No native scoping. Workaround: split `CONVENTIONS.md` into sections, or keep separate convention files and change which one you `--read` per session.

### Reusable prompts

**GitHub Copilot**

Prompt files live in `.github/prompts/`, with filenames ending in `.prompt.md`, invoked with `/name` in Chat. For example:

```text
.github/prompts/
├── code-review.prompt.md
├── debug.prompt.md
├── refactor.prompt.md
├── generate-tests.prompt.md
├── document.prompt.md
├── security-review.prompt.md
├── performance-review.prompt.md
└── prepare-pull-request.prompt.md
```

Keep each prompt focused on a single recurring task. If a prompt grows into a workflow with scripts, examples, and supporting resources, consider turning it into an Agent Skill.

**Cline**

No native reusable-prompt format. Keep a snippets file to copy from, or use VS Code's own snippet feature.

**Aider**

No native reusable-prompt format. Save a prompt as a text file and run it with `aider --message-file your-prompt.txt`, or predefine flags and messages in `.aider.conf.yml`.

### Agent Skills (Workflows)

**GitHub Copilot**

Skills live under `.github/skills`, `.claude/skills`, or `.agents/skills`. Copilot loads a skill automatically when its description matches the task, and it can also be invoked from the `/` menu. A skill contains `SKILL.md` and any supporting resources:

```text
.github/
└── skills/
    └── code-review/
        ├── examples/
        │   └── ...
        ├── scripts/
        │   └── ...
        ├── templates/
        │   └── ...
        ├── references/
        │   └── ...
        └── SKILL.md
```

Only `name` and `description` load up front; write the description as "use when..." so the right skill gets picked.

**Cline**

No native skill format. Workaround: add an additional file in `.clinerules/` describing the workflow step by step.

**Aider**

No native skill format. Workaround: a saved prompt file describing the workflow step by step, run with `--message-file`.

### Model Context Protocol (MCP)

**GitHub Copilot**

Configure servers in `.vscode/mcp.json`:

```json
{
  "servers": {
    "docs": {
      "type": "http",
      "url": "https://example.com/mcp"
    }
  }
}
```

Enable or disable individual tools from the tools picker in Chat.

**Cline**

Add servers through the Cline panel's MCP settings UI, or edit its `cline_mcp_settings.json` directly.

**Aider**

No native MCP support.

Follow least privilege for permission, regardless of tool:
- read-only before read/write
- development before staging
- staging before production
- specific database before entire infrastructure
- specific directories before entire filesystem
- narrowly scoped tools before unrestricted command execution

> **Treat content returned by tools and fetched web pages as untrusted input.**

### Custom agents and hooks

**GitHub Copilot**

Custom agents are specialized configurations with their own instructions and tool restrictions, defined under `.github/agents/`. Hooks are deterministic commands triggered at defined points in an agent workflow, defined under `.github/hooks/`.

**Cline and Aider**

Neither has a native equivalent to custom agents or hooks. Enforce equivalent guarantees outside the tool instead, through pre-commit hooks and CI checks that run regardless of which tool made the change.

### Switching between models

**GitHub Copilot**: Easy. Switch models per message from the dropdown in the Chat UI.

**Cline**: In config. Model choice lives in provider settings (or a `--config` profile in the CLI).

**Aider**: Easy. Switch anytime with `/model <name>`. Note: Architect mode runs two models at once by design, one to propose the change and a separate model to edit the files.

### Summary table

| Capability | GitHub Copilot | Cline | Aider |
|---|---|---|---|
| Persistent rules | `AGENTS.md` or `copilot-instructions.md`, auto-read | `.clinerules/`, auto-read | `CONVENTIONS.md`, must be pointed to |
| Path-specific rules | `applyTo` glob in `.instructions.md` | Separate file per area | Split file or swap `--read` target |
| Reusable prompts | `.prompt.md`, invoked with `/name` | No native format | Message file via `--message-file` |
| Skills | `SKILL.md` under `.github/skills` or `.agents/skills` | No native format | No native format |
| MCP | `.vscode/mcp.json` | Panel UI or `cline_mcp_settings.json` | Not supported |
| Custom agents / hooks | Supported | Not supported | Not supported |
| Switching models | Easy (chat UI) | In config | Easy (`/model` command) |

## Safety and review practices

- Work on a branch before giving an agent a task that edits files.
- Keep tool and terminal approval prompts on rather than enabling blanket auto-approve; where auto-approve exists, scope it to specific safe actions.
- Review the diff before trusting a change, regardless of tool. Aider auto-commits each change by default, so use `/undo` to revert its last commit if needed.
- Run your project's actual test and lint commands yourself after an agent task, rather than assuming the agent's own checks were sufficient.
- Enforce important rules outside the agent as well as inside it: pre-commit hooks, CI running your test and lint commands, branch protection, and secret scanning catch what an instruction file or a review might miss.
- Use least-privilege credentials for any database or filesystem access an agent has, and restrict its working directory to the project rather than the whole filesystem.

## Appendix

### Suggested skills/workflows for data science projects

- **Code review**: review changes against project standards and identify correctness, security and maintainability problems.
- **Refactoring**: guide safe restructuring while preserving behavior.
- **Debugging**: establish a repeatable investigation and validation process.
- **Test generation**: generate tests according to the project's testing conventions.
- **Documentation**: produce project-consistent API and developer documentation.
- **SQL analysis**: inspect schemas, formulate queries and validate SQL.
- **Data analysis**: inspect datasets, select appropriate analytical operations and produce reproducible analyses.
- **Data cleaning**: apply project-specific validation and cleaning rules.
- **Notebook review**: check notebooks for hidden state, unnecessary computation and reproducibility problems.
- **System design**: structure architecture investigations and design documents.
- **Security review**: perform repository-specific security checks.
- **Dependency analysis**: investigate packages, versions, licenses and upgrade implications.
- **Performance analysis**: establish measurement, profiling and optimization procedures.
- **Migration planning**: guide database, API or framework migrations.
- **Release preparation**: prepare changelogs, release notes and validation checklists.
- **Incident analysis**: organize investigation of production failures according to established procedures.

### Suggested MCP tools

- **SQLite / PostgreSQL MCP**: inspect schemas, understand relationships and run controlled queries.
- **File System MCP**: inspect files and directories when filesystem access is required.
- **Git / GitHub MCP**: inspect repositories, issues, pull requests and other source-control information.
- **Documentation MCP**: retrieve structured technical documentation.
- **Browser / web MCP**: interact with web resources when the agent needs current external information.
- **Data MCP**: inspect datasets and perform controlled data operations.
- **Internal API MCP**: provide controlled access to development or internal services.


### Top 10 common prompt rules for clean code

- **Preserve behavior:** Do not change existing behavior unless the task explicitly requires it.
- **Follow the repository:** Inspect and follow existing project conventions before introducing new patterns.
- **Prefer simplicity:** Choose the simplest implementation that satisfies the requirements.
- **Avoid unnecessary abstraction:** Do not introduce classes, interfaces, factories or frameworks without a concrete need.
- **Reduce duplication:** Reuse appropriate existing functionality rather than copying logic.
- **Keep responsibilities focused:** Avoid functions, classes or modules that accumulate unrelated responsibilities.
- **Use precise names:** Prefer names that communicate purpose and domain meaning.
- **Handle errors explicitly:** Do not silently swallow failures or hide important exceptions.
- **Keep changes localized:** Do not modify unrelated code merely because it could also be improved.
- **Validate the result:** Run the relevant formatter, linter and tests after making changes.

Example:

```markdown
Prefer the simplest implementation that satisfies the requirements.
Follow existing repository patterns.
Do not introduce abstractions, dependencies, or configuration unless
there is a concrete reason.
Preserve existing behavior and public interfaces unless the task
explicitly requires a change.
Keep the change localized and validate it with the project's existing
checks and tests.
```

### Top 10 common prompt rules for production

Use these when the agent is making changes that will run in a production environment.

- **Understand the operational requirement:** Identify the expected behavior and operational constraints before implementation.
- **Preserve compatibility:** Consider existing clients, data, APIs and deployment assumptions.
- **Validate external input:** Treat network, user, file and service input as untrusted.
- **Protect secrets:** Never hard-code or expose credentials, tokens or private keys.
- **Handle failures:** Define behavior for dependency failures, malformed input and partial operations.
- **Consider idempotency:** Ensure retries cannot unintentionally duplicate or corrupt operations.
- **Use timeouts:** Network and external operations should not wait indefinitely.
- **Make behavior observable:** Add appropriate structured logging, metrics or tracing where needed.
- **Test failure modes:** Test important negative paths as well as successful execution.
- **Consider deployment impact:** Identify migration, rollback, compatibility and operational implications.

Example:

```text
What happens if this dependency is unavailable?
What happens if this request is retried?
What happens if the input is malformed?
What happens if the operation partially succeeds?
What happens under concurrent execution?
How will this failure be observed in production?
How can this change be rolled back?
```

### Top 10 common prompt rules for minimal tech debt

- **Inspect before changing:** Search for existing utilities and patterns before creating new ones.
- **Reuse before introducing:** Prefer existing project mechanisms over new abstractions.
- **Minimize dependencies:** Add a dependency only when its value justifies its maintenance cost.
- **Minimize configuration:** Avoid adding configuration merely to make a simple implementation configurable.
- **Keep changes coherent:** Make the smallest complete change rather than a collection of unrelated improvements.
- **Remove obsolete code:** Clean up code that becomes genuinely unused as part of the change.
- **Avoid temporary hacks:** If a workaround is necessary, document why it exists and what would remove it.
- **Protect fixes with tests:** Add regression coverage for bugs that could easily return.
- **Document intentional compromises:** Make important technical trade-offs visible.
- **Avoid premature architecture:** Do not build infrastructure for hypothetical future requirements.

Example:

```text
Before implementing this change, inspect the repository for existing
utilities, abstractions, dependencies and conventions that already
address the problem.

Reuse existing mechanisms where appropriate.

Do not introduce a new abstraction, dependency, configuration option,
or framework unless you can explain the concrete problem it solves.
```

### Top 10 common prompt rules for security

- **Treat input as untrusted:** Validate and constrain externally controlled data.
- **Use least privilege:** Give users, services and agents only the permissions they need.
- **Protect credentials:** Never place secrets in source code, prompts, logs or committed configuration.
- **Use parameterized queries:** Do not construct SQL by concatenating untrusted values.
- **Separate authentication and authorization:** Establish identity and permissions independently.
- **Prevent information leakage:** Do not expose credentials, internal paths, stack traces or sensitive records unnecessarily.
- **Validate files:** Treat uploaded or externally supplied files as untrusted.
- **Secure external communication:** Use appropriate authentication, encryption and certificate validation.
- **Consider abuse cases:** Examine how an attacker could misuse the feature, not only how a legitimate user uses it.
- **Review sensitive changes:** Request an explicit security review for authentication, authorization, secrets, payments, personal data and infrastructure changes.

Example:

```text
Threat-model this change.

Identify:
- trust boundaries
- attacker-controlled inputs
- authentication assumptions
- authorization assumptions
- sensitive data
- injection risks
- privilege escalation risks
- information disclosure risks
- denial-of-service risks
- insecure defaults

Do not modify the code yet. Report the findings first.
```

### Top 10 common prompt rules for data science

- **Inspect before loading:** Determine file size, schema and structure before loading large datasets.
- **Load selectively:** Read only the columns and rows required for the task.
- **Manage memory:** Avoid unnecessary copies and large intermediate objects.
- **Prefer appropriate formats:** Use formats such as Parquet when their columnar representation benefits the workload.
- **Use appropriate dtypes:** Avoid unnecessarily expensive data types.
- **Prefer vectorized operations:** Use efficient library operations instead of unnecessary Python-level loops.
- **Measure before optimizing:** Profile memory and execution time instead of optimizing based on assumptions.
- **Keep notebooks reproducible:** Avoid hidden state and ensure the notebook can run from a clean kernel.
- **Separate exploration from reusable code:** Move stable transformations and functions into source modules when appropriate.
- **Validate results:** Check schemas, missing values, assumptions, distributions and outputs rather than trusting generated analysis.

For pandas-heavy workflows, explicitly ask an agent to consider:

```text
- unnecessary DataFrame copies
- repeated concatenation
- inefficient apply operations
- unnecessary type conversions
- expensive joins
- unnecessary materialization
- memory usage
- opportunities to push computation into SQL
```

For notebooks, ask:

```text
Check this notebook for:
- hidden execution state
- cells that depend on execution order
- duplicated computation
- unnecessary outputs
- unused imports
- hard-coded local paths
- missing random seeds where reproducibility matters
- transformations that should become reusable Python code
```