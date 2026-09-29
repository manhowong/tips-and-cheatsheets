# Quickstart for AI-Assisted Development

This guide covers three tools: Copilot Chat and Cline (both VS Code extensions), and Aider (a CLI tool).

(As of Sep 2026)

| | Copilot Chat | Cline | Aider |
|---|---|---|---|
| Interface | IDE extension | IDE extension | Terminal (CLI) |
| Modes | Ask, Edit, Agent | Plan, Act | Code, Ask, Architect |
| Inline autocomplete | ✔️ (GitHub-hosted models) | ❌ | ❌ |
| BYOK (own API key) | ✔️ | ✔️ | ✔️ |
| Local models | ✔️ (via BYOK) | ✔️ | ✔️ |
| No signup | ✔️ (Except autocomplete) | ✔️ | ✔️ |
| Switching models | Easy (chat UI) | In config | Easy (`/model` command) |
| Config format | `.github/`, `AGENTS.md` | `.clinerules/` | `CONVENTIONS.md`, `.aider.conf.yml` |


## Basic setup

**1. Install the tool.**
- **Copilot:** Install the GitHub Copilot Chat extension in VS Code.
- **Cline:** Install the Cline extension in VS Code.
- **Aider:** Run `pip install aider-install && aider-install` (or `pipx install aider-chat`).

**2. Authenticate / connect a model.**
- **Copilot:** Sign in with a GitHub account on a Copilot plan for autocomplete. Chat alone can run BYOK without sign-in (see Advanced configurations).
- **Cline:** Get an API key from a provider (Anthropic, OpenAI, etc.) and enter it in Cline's settings panel.
- **Aider:** Set the provider's API key as an environment variable (e.g. `export ANTHROPIC_API_KEY=...`).

**3. Open your project (git repository).**
- **Copilot/Cline:** Open the project folder in VS Code.
- **Aider:** `cd` into the repo in a terminal.

**4. Create a rules file.**
- **Copilot:** Create `AGENTS.md` at the repo root (or run `/init` in Chat to generate one from your codebase).
- **Cline:** Create a `.clinerules/` folder with one or more `.md` files.
- **Aider:** Create `CONVENTIONS.md`.

**5. Point the tool at that file.**
- **Copilot:** Nothing to do (`AGENTS.md` and `.github/copilot-instructions.md` are read automatically).
- **Cline:** Nothing to do (everything in `.clinerules/` is read automatically).
- **Aider:** Add `read: CONVENTIONS.md` to `.aider.conf.yml`, or pass `--read CONVENTIONS.md` each run.

## Usage

**1. Open the tool's chat interface.**
- **Copilot:** Open Chat with `Ctrl+Alt+I` (`⌃⌘I` on macOS).
- **Cline:** Click the Cline icon in the VS Code sidebar.
- **Aider:** Run `aider` in the terminal.

**2. Optional: Switch to agent mode.**
- **Copilot:** Select **Agent** in the mode picker.
- **Cline:** Select **Act** mode (use **Plan** mode first if you want it to propose an approach before editing).
- **Aider:** No separate mode: it edits by default.

**3. Optional: Run a read-only prompt for testing.**

**4. Create a branch for real changes.** (`git checkout -b <branch-name>`)

**5. Set approval behavior.**
- **Copilot:** Leave tool/terminal approval prompts on rather than enabling auto-approve.
- **Cline:** In settings, leave auto-approve off, or restrict it to safe actions like reading files.
- **Aider:** Don't pass `--yes`; approve each proposed edit interactively.

**6. Give it a real task.**

**7. Review the diff.** (`git diff`)
- **Aider**: `/undo` to revert its last commit if needed.

**8. Run tests and linter.** 
- Run your project's actual test/lint commands rather than assuming the agent's own checks were sufficient.

**9. Commit the change.**

### Good tasks for AI agents

- Exploring, inspecting, or reviewing codebase:
  - explain unfamiliar code
  - inspect repository conventions
  - locate relevant code in a repository
  - investigate dependencies
- Agile implementation:
  - generating boilerplate
  - implementing narrowly defined features
- Testing or debugging:
  - generate tests
  - debug failures
  - identify security concerns
- Maintenance work:
  - write documentation
  - refactor existing code
  - reviewing diffs
  - prepare pull requests

### Prompt design

A useful prompt normally contains:
- Goal
- Relevant context
- Constraints
- Expected behavior
- Files or symbols involved
- Validation requirements

Example:

```text
Find the cause of the failing authentication test.

Relevant files:
- src/auth/service.py
- tests/auth/test_service.py

Constraints:
- Preserve the public API.
- Do not change authentication semantics.
- Do not add dependencies.

Expected behavior:
- Invalid credentials should return the existing authentication error.
- Valid credentials must continue to work.

Please inspect the relevant code first, identify the root cause,
make the smallest coherent change, add a regression test, and run
the relevant tests.
```

## Advanced configurations

(See `AI-assisted-dev-customization.md` for more detailed customization.)

**Add reusable, multi-step workflows (skills).**
- **GitHub Copilot:** Create `.github/skills/<name>/SKILL.md` (also checks `.agents/skills`) with `name` and `description` frontmatter.
- **Cline:** No native skill format (approximate with an additional file in `.clinerules/` per workflow).
- **Aider:** No native skill format (save the workflow as a text file and run it with `aider --message-file your-prompt.txt`).

**Add reusable prompt shortcuts.**
- **GitHub Copilot:** Create `.github/prompts/<name>.prompt.md`; run with `/<name>` in Chat.
- **Cline:** No built-in shortcut system (keep a snippets file to paste from, or use VS Code snippets).
- **Aider:** Add default flags/messages to `.aider.conf.yml`, or make a shell alias for common invocations.

**Add path-specific rules.**
- **GitHub Copilot:** Create `.github/instructions/<name>.instructions.md` with an `applyTo` glob in frontmatter.
- **Cline:** Add a separate file inside `.clinerules/` per area of the codebase.
- **Aider:** Split `CONVENTIONS.md` into sections, or keep separate files and change which one you `--read` per session.

**Connect external tools and data (MCP).**
- **GitHub Copilot:** Create `.vscode/mcp.json` with a `servers` block.
- **Cline:** Add servers from the Cline panel's MCP settings UI.
- **Aider:** No native MCP support.

**Enforce rules outside the agent.**
- Add pre-commit hooks (format, lint) and a CI job that runs your test/lint command, so agent-introduced issues get caught even if a review misses them.

**Restrict write access where it matters.**
- Use read-only database credentials by default, and keep the agent's working directory scoped to the project rather than the whole filesystem.

**Keep multiple tools' rules in sync.**
- Treat `AGENTS.md` as the source of truth, and copy or lightly adapt its content into `.clinerules/` and `CONVENTIONS.md` whenever you update it, since Cline and Aider don't read `AGENTS.md` automatically.
