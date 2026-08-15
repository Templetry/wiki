# `piece.yml` specification — v1

**Status:** normative for engine v1.4+ (ADR-0014, [study VII](../study/pieces-v1.md)). A piece is a decoupled unit a generated project adopts after creation, with its own variables and drift anchor.

## Shape

A template-local piece lives inside its form at `pieces/<name>/`; everything in the directory except `piece.yml` **is** the piece.

```yaml
schema_version: 1            # required, must be 1
name: axios-api              # required, kebab-case
description: Typed axios API client under src/api

variables:                   # sub-customization, same rules as the manifest's
  - key: api_base
    label: API base URL
    default: /api

patches:                     # the ONLY way to touch existing project files
  - file: package.json       # .json / .yml / .yaml / .toml targets
    op: add                  # add | replace | remove
    path: /dependencies/axios
    value: "^1.7.0"
```

## Semantics

- **Identity**: piece files are written in the **form's canonical identity**; on apply they render through the form manifest's identity map expanded with the project's recorded variables (from the answers file). A piece declares no identity of its own.
- **Directives**: `tpl:var` works against both the project's and the piece's variables. `tpl:if` is an error inside a piece — a piece is its own unit; conditional content within it is a smell.
- **Enforced decoupling**: a rendered piece path that already exists in the project is an **error** and nothing is written. Wiring into shared files happens exclusively through `patches`.
- **Variable namespaces**: a piece variable key that collides with a template variable key is an error.
- **Recording**: applying a piece appends to the answers file's `pieces:` list — name, source, resolved commit (its own drift anchor), resolved variables and the owned file list (see [answers-file.md](answers-file.md)).
- **Re-adding** an applied piece is an error; removal is not in v1 (the recorded file list keeps it possible later).
- **Updates** (engine v1.8+): `templetry update` re-renders every applied piece at the template's head with its recorded inputs and puts the result through the same diff and three-way merge as the template's own files. Entries report which piece owns them, and applying moves the template's and every piece's anchor forward together. The answers file itself is never merged — it is rewritten from the record, so the `pieces:` list and the recorded inputs always survive.

## Common pieces (v1.9+, ADR-0016)

A piece may live outside any form, in a shared repository listed in the registry's top-level `pieces` array. Such a piece declares what it supports:

```yaml
schema_version: 1
name: audit-trail
description: Append-only record of who changed what and when — Go + SQLite implementation
applies_to:            # template names (the `name` of their template.yml)
  - go-rest-sqlite     # omit entirely for a universal piece
```

Rules:

- **Form-local wins.** If a form ships a piece with the same name, that one is used.
- **One name, many implementations.** Several directories may declare the same `name` with disjoint `applies_to`; the project asks for the name and gets the implementation for its ecosystem. An incompatible request fails with `piece X does not apply to Y`.
- **The source is recorded per piece.** Adopting a common piece writes *its* repository into the answers file, and `templetry update` fetches it from there — fix the piece once, every project that adopted it gets the fix.

## Known boundary (v1) and the socket pattern

Patches only touch structured formats, so ecosystems whose wiring lives in code files need the form to be authored **piece-ready** with a *socket*: a registration point the piece plugs into from a file it owns, so no existing file is edited.

Proven in `go/http-service` (2026-08-13): the form exposes

```go
var registrars []func(*http.ServeMux)
func Register(f func(*http.ServeMux)) { registrars = append(registrars, f) }
// NewMux ranges over registrars after its own routes
```

and the `version-endpoint` piece ships a file whose `init` calls `api.Register(...)`. The form compiles with zero pieces (empty slice) and gains routes as pieces arrive — CI verifies both states. The same shape applies elsewhere: a `pieces/` package the app imports and iterates, a conditional `apply(from = ...)` in Gradle whose target the piece owns, an auto-included routers directory in FastAPI.
