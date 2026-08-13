# Study VII — Lazy pieces

**Date:** 2026-08 · **Status:** study — feeds ADR-0014

**Owner's requirement (roadmap lane F):** a template can offer *pieces* — fully decoupled units of code you may not want at creation time but can bring in later, with update capability and their own sub-customization; choosable at creation or added after the project exists; distinguishing pieces of one template from pieces common to several.

## Concept

| | Feature | **Piece** |
|---|---|---|
| When | render time only | at creation **or any time later** |
| Mechanism | toggles content baked into the base tree | adds a decoupled unit to a living project |
| Lifecycle | none (part of the render) | own version anchor, updates independently |
| Sub-customization | none (shares template variables) | own variables, recorded per piece |

**Anti-explosion rule inherited from ADR-0011:** anything expressible as a feature must stay a feature. A piece must justify its independent lifecycle.

## The design

### A piece is a directory + `piece.yml`

Template-local pieces live in the form repo under `pieces/<name>/`:

```
android/single-module/
  template.yml
  app/…
  pieces/
    room/
      piece.yml
      app/src/main/kotlin/es/sebas1705/androidsingleapp/data/…
```

```yaml
# piece.yml
schema_version: 1
name: room
description: Room persistence layer
variables:                       # sub-customization, recorded per piece
  - key: db_name
    default: app
patches:                         # the ONLY way to touch existing files
  - file: gradle/libs.versions.toml
    op: add
    path: /libraries/room-runtime
    value: { module: "androidx.room:room-runtime", version.ref: "room" }
```

- **No `files` list**: everything in the piece directory except `piece.yml` *is* the piece.
- **No `identity` section**: a template-local piece is written in its form's canonical identity; the engine renames it with the identity map of the form manifest plus the project's recorded variable values (from the answers file) plus the piece's own variables.
- Directives: `tpl:var` against piece variables; full `tpl:if` against features postponed (a piece is its own unit — conditional content inside it is a smell).

### Decoupling is enforced, not hoped for

Applying a piece writes its rendered files into the project **only if none of those paths already exist** — a collision is an error naming the offending path. Wiring into shared files (settings.gradle includes, version catalogs, package.json) happens **exclusively through declared patches**. Consequences:

- **No three-way merge on add** — adds are trivial and safe.
- **Piece-vs-piece ordering is irrelevant for files** (they cannot collide with anything, including each other); patches apply in add order, which the answers file records.
- The unit stays honest: if a piece "needs" to rewrite a project file, it is not a piece — it is a feature or a template change.

### The answers file grows a `pieces:` section (additive)

```yaml
pieces:
  - name: room
    source: github.com/Templetry/android@main/single-module/pieces/room
    commit: <sha resolved at add time>
    variables: {db_name: app}
    files: [gradle/…, app/src/…]     # what the piece owns on disk
```

- `commit` is the piece's **own drift anchor**: `templetry update` re-renders the piece at head with the recorded inputs and runs the same diff/three-way machinery as template updates (lane A's package) — pieces update independently of the base template.
- `files` records ownership, mirroring the ADR-0005 wisdom: **removal is not in v1, but recording the file list from day one keeps the door open** (a future `remove` deletes owned files; reversing patches remains the researched part).

### Discovery needs no registry change (v1)

Pieces ship inside the form repo, so `templetry pieces` lists them by fetching the form tarball — **zero registry schema change**. The form render excludes `pieces/**` (like `template.yml` itself); the form still compiles with the directory present because nothing wires it in.

### Common pieces are phase 2

A common piece (shared across templates) needs what template-local pieces get for free: its own canonical identity map, a compatibility declaration (which parents/forms or capability tags), and a discovery channel (registry v2.1, additive `pieces` entry). Per-ecosystem reality (a Room piece only means something on Android/KMP) suggests common pieces will mostly be **per-parent**, which the compat declaration expresses. Deferred until template-local pieces have proven the lifecycle.

## Resolved open questions

| Question (roadmap) | Resolution |
|---|---|
| Piece-vs-piece ordering/conflicts | Files can't conflict (existence check); patches apply in recorded add order |
| Sub-customization casings colliding with project identity | Piece variables render through the same `Expand`; the planner's longest-first replacement order and the existing collision detection apply unchanged |
| Can a piece be removed? | Not in v1; the recorded `files:` list keeps it possible later |
| Common pieces per ecosystem | Phase 2, compat-declared, mostly per-parent |
| CI cost | One extra job per piece (render base → add piece → compile); linear, not combinatorial, because pieces are decoupled by construction |

## Engine surface (all additive)

- New package `piece/`: load/validate `piece.yml`, `Apply(projectDir, …)`, and the answers `pieces:` read/write.
- Planner: exclude `pieces/**` from form renders.
- CLI: `templetry pieces [dir]` (list what the project's template offers, marking applied ones) · `templetry add <piece> [dir] [--set k=v]` · `templetry update` grows piece awareness.
- MCP: `list_pieces`, `add_piece` tools; update tool reports pieces too.
- Desktop: pieces panel in Local's project preview (list/add/update) — after the engine lands.

## Verification plan

1. Unit: piece load/validate, collision refusal, answers round-trip.
2. A real piece on `web/react-spa` (npm-buildable locally end to end): render base → `add` → build → mutate template → `update` picks the piece.
3. A real piece on `android/single-module` (`room`) verified by its parent CI job.
