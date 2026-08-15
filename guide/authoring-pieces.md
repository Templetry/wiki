# Authoring pieces

A **piece** is a decoupled unit a living project adopts after creation. Writing one is mostly a design exercise: the code is the easy part, the decoupling is the work.

Normative reference: [`piece.yml` spec](../spec/piece-yml.md) · rationale: [ADR-0014](../adr/0014-lazy-pieces.md), [ADR-0016](../adr/0016-common-pieces.md).

## Is it a piece?

| If the variation is… | It is a… |
|---|---|
| Additive, decided at creation, dissolves into the project | **feature** |
| Structural — a different project shape | **form** |
| Independent, adoptable later, with its own lifecycle | **piece** |

If it must be decided before the first commit, it is not a piece.

## Where it lives

Inside its form:

```
fastapi-users/
  template.yml
  pieces/rbac/          # or _pieces/rbac/ — see below
    piece.yml
    <everything else is the piece>
```

Use **`_pieces/`** when the template's own toolchain globs the source tree — Go's `./...`, a Gradle source set, a `tsconfig` include. Piece files reference symbols of the package they will be applied into, so a scanned `pieces/` directory breaks the template's own build. The leading underscore makes Go and most toolchains skip it. Both roots are supported; prefer `_pieces/` unless you have a reason not to.

## The manifest

```yaml
schema_version: 1
name: audit-trail
description: Append-only record of who changed what and when

variables:                  # optional sub-customization
  - key: table_name
    label: Table name
    default: audit_log

patches:                    # the ONLY way to touch existing project files
  - file: pyproject.toml
    op: add
    path: /project/dependencies/-
    value: "sqlalchemy>=2.0"
```

Rules that differ from templates:

- **A piece declares no identity map.** It is written in the form's canonical identity and renders through the *project's* recorded variables, so it lands already renamed.
- **`tpl:var` works**, against both the project's and the piece's variables. **`tpl:if` is an error** — a piece is one unit; conditional content inside it means you have two pieces.
- A piece variable key that collides with a template variable key is an error.

## The rule that shapes everything: no collisions

**A rendered piece path that already exists is an error, and nothing is written.** That is not a limitation to work around; it is the guarantee that lets a user adopt a piece without reading it first.

So the piece must bring only *new* files, and reach existing ones only through declared `patches` — which cover `.json`, `.yml`/`.yaml` and `.toml`.

## Wiring into code: the socket pattern

Patches do not touch code files, so a piece that needs to register itself requires the **form** to be authored piece-ready, exposing a registration point the piece plugs into from a file it owns.

Proven in `go/http-service`:

```go
// in the form
var registrars []func(*http.ServeMux)
func Register(f func(*http.ServeMux)) { registrars = append(registrars, f) }
// NewMux ranges over registrars after its own routes
```

```go
// in the piece, a file the piece owns
func init() { api.Register(func(mux *http.ServeMux) { /* … */ }) }
```

The form compiles with zero pieces (empty slice) and gains behaviour as pieces arrive. **CI must verify both states** — the bare form and the form with pieces applied — or the socket rots.

Equivalent shapes elsewhere: a `pieces/` package the app imports and iterates, a conditional `apply(from = …)` in Gradle whose target file the piece owns, an auto-included routers directory in FastAPI.

## Pieces per object

Give the piece a variable that names the thing, and the identity follows:

```sh
templetry add crud-resource ./my-api --set entity=Product
```

lands `models_product.py`, `routers/product.py`, `class Product`. Adopt it once per entity; each application is recorded separately. This is the piece equivalent of a scaffold generator, except it stays updatable.

## Common pieces

A piece that is not specific to one form lives in [Templetry/pieces](https://github.com/Templetry/pieces) and declares what it supports:

```yaml
schema_version: 1
name: audit-trail
description: Append-only record — Go + SQLite implementation
applies_to:
  - go-rest-sqlite       # omit entirely for a universal piece
```

- **Form-local wins**: a same-named piece inside the form takes precedence.
- **One name, many implementations**: several directories may declare the same `name` with disjoint `applies_to`. The user asks for the name and gets their ecosystem's implementation; an incompatible request fails with `piece X does not apply to Y`.
- The piece records **its own** repository as its source, so a fix upstream reaches every project that adopted it through `templetry update`.

Register it in the registry's top-level `pieces` array (schema v2.1):

```json
"pieces": [
  { "name": "audit-trail", "repo": "Templetry/pieces", "ref": "main", "path": "audit-trail-go" }
]
```

## Checklist before publishing

- [ ] The form still builds with the piece **absent**.
- [ ] The form builds with the piece **applied** — verified in CI, not by hand.
- [ ] No rendered path collides with anything the form produces.
- [ ] Every shared file it touches goes through `patches`.
- [ ] Its description says what it expects — pieces cannot depend on pieces.
- [ ] Its tests do not hardcode a value a variable can change.

## Limits (today)

Removal is not implemented; pieces cannot declare dependencies on other pieces. Both are recorded as known gaps in the spec, and the answers file already stores what a future removal would need.
