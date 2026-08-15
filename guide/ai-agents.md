# AI agents and MCP

Templetry is deliberately agent-shaped. A template declares its own inputs in machine-readable form, rendering is deterministic, and every generated project carries a provenance record — so an agent can discover what a template takes, scaffold with it, and come back later to update it, without a human translating anything.

Two independent mechanisms serve this: **the MCP server** (agents drive Templetry) and **`AGENTS.md`** (Templetry instructs agents).

## The MCP server

`templetry-mcp` ships in the same [releases](https://github.com/Templetry/engine/releases) as the CLI. It speaks the Model Context Protocol (JSON-RPC 2.0 over stdio) with no dependencies and no configuration.

```sh
claude mcp add templetry -- templetry-mcp
```

Any MCP client works the same way — the server is a plain executable reading stdin and writing stdout.

### The seven tools

| Tool | What it does |
|---|---|
| `list_templates` | Every parent and form in the catalog, with status and description |
| `get_form_schema` | A form's variables (patterns, options, defaults) and features, as JSON |
| `plan` | Dry run: exactly what rendering would produce — writes nothing |
| `render` | Render into a directory, with its `.templetry-answers.yml` |
| `update` | The update cycle on an existing project; previews unless `apply` is set |
| `list_pieces` | The pieces a project's template offers, marking applied ones |
| `add_piece` | Apply one piece, with its own variables |

All of them take an optional `registry` so an agent can work against a private catalog.

### The loop that works

```
list_templates → get_form_schema → plan → render
```

`get_form_schema` is the step agents skip and should not: it returns the variable patterns, so the agent can construct valid inputs instead of guessing and retrying. `plan` then shows what will happen before anything is written — a free dry run.

For an existing project: `list_pieces` → `add_piece`, or `update` without `apply` to see the diff, then with `apply`.

### Safety properties an agent can rely on

- **`plan` and preview-mode `update` write nothing.** They are always safe to call.
- **`render` refuses a non-empty directory** unless `force` is set.
- **`update` never deletes.** Applying only adds and rewrites; user-added files are untouched.
- **`add_piece` never overwrites.** A collision aborts the whole operation.
- **Rendering is deterministic**: same template commit, same inputs, byte-identical output. An agent can re-render to compare rather than remember.

## AGENTS.md

Every Templetry template ships an `AGENTS.md`, and it renames with the project like every other file — so the generated project is born with an operating contract for whatever agent works on it.

What belongs in it:

- **Mission** — one paragraph on what this project is.
- **Core rules** — the conventions that are not negotiable here.
- **Checks that must pass** — the exact commands, so "done" is verifiable rather than claimed.
- **The safe change workflow** — where to add code, what never to edit by hand.

Keep it imperative and short. Agents follow lists; they skim prose.

Repositories in the organization carry the same file for their own contributors, human or otherwise — the engine's `AGENTS.md` states the layering rules, the catalog's states the manifest and CI conventions.

## Why the manifest matters here

An agent that has to infer a template's inputs from its source is guessing. Because knowledge lives in the manifest ([ADR-0002](../adr/0002-knowledge-lives-in-the-manifest.md)), `get_form_schema` returns the same declaration the desktop builds its form from and the CLI validates against. One source of truth, three consumers — and the machine-readable one is not an afterthought bolted on for agents.
