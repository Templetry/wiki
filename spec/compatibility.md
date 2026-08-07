# Compatibility policy — v1

**Status:** normative from engine v1.0.0 (draft until then — see [study VI](../study/road-to-v1.md)). This document defines the promise the version number makes.

## The public API

Semver's "public API" means, for Templetry, exactly these surfaces:

| Surface | Spec | Promised to |
|---|---|---|
| `template.yml` schema (`schema_version: 1`) | [template-yml.md](template-yml.md) | template authors |
| `tpl:` directive grammar and comment-style table | [directives.md](directives.md) | template authors |
| `.templetry-answers.yml` (`schema_version: 1`) | [answers-file.md](answers-file.md) | every generated project — the update cycle depends on it |
| `registry.json` schema v2 | ADR-0011 | catalog authors |
| CLI surface: command names, flags, exit codes, `--json` output shapes | engine README | scripts and CI pipelines |
| The engine's exported Go API | godoc | embedders (the desktop app) — covered by construction: Go modules force `/v2` on any breaking change |

## Explicitly not public API

- **Human-readable CLI output** (plan text, progress lines) — `file:line` positioning of errors is kept, wording may change.
- **Error message texts.**
- **The official catalog's content** — templates evolve freely, versioned by their own git refs; the answers file pins the exact commit, so existing projects never break retroactively.
- **The wiki** — documentation, not contract.

## Change rules

- **Additive → minor**: new optional manifest keys, new casings, new directives, new CLI commands/flags, new registry fields. A v1.x engine renders every valid v1.0 template.
- **Breaking → major**, always paired with a `schema_version` bump on the affected file format and written migration notes. Go's `/v2` import-path cost is accepted as the deliberate price of breaking.
- **Deprecation**: announced in the CHANGELOG at least one minor release before the behavior changes; removal only at a major.
- **Support window**: the latest minor is supported (single-maintainer project — no backports).

## Version relationships

- Engine version = Go module version = CLI version (one artifact).
- The desktop app versions independently and displays both its own and the embedded engine's version.
- Rendered projects depend on neither: the answers file records template source + commit, so any future engine can re-render and update them.
