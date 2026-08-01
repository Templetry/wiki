# Study IV — Template forms: parents, forms and combinability

**Date:** August 2026 · **Status:** study for ADR-0011

Requirement (project owner): each catalog *parent* (e.g. KMP) should offer multiple **forms** — all-platforms vs some, modular vs single-module, `core/data/domain/feature` vs `core/data/domain/ui` — and those variations should be combinable.

## The tension

Combinability collides with the compile principle (ADR-0003). Two kinds of variation exist:

1. **Additive variation** — add/remove files and blocks over one skeleton (a platform target, Room, auth). Expressible with the existing feature mechanism (`files` globs + `tpl:if` + patches). Freely combinable; the template still compiles with everything on.
2. **Structural variation** — the *same* code arranged differently (multi-module vs single-module; `feature/*` vs `ui/` layers). Not expressible by adding/removing: parallel trees inside one template would stop compiling (breaks ADR-0003), and programmatic tree transforms are the rejected Yeoman school (ADR-0001/0002). Free combination of structural axes also explodes maintenance: platforms × modularity × architecture = dozens of trees for one maintainer (NFR5).

## The model: parent → form → features

| Level | What it is | Combinable? |
|---|---|---|
| **Parent** | Catalog concept (`kmp`, `android`) — not a repo | — |
| **Form** | One *structural* variant; its own repo; compiles on its own with its own CI | chosen, not combined |
| **Feature** | Additive variation inside a form (targets, capabilities) | freely combinable at render |

**Golden rule:** additive → feature; structural → form.
**Anti-explosion rule:** a new form must justify itself structurally — if it can be expressed as features on an existing form, it must be.
**Versions:** free via git — each form's `ref`/tags; the answers file records which one was rendered.

## Applied to today's catalog

- Parent `kmp`: forms `modular-base` (= kmp-native-base) and `single-module` (= compose-multiplatform-app). The overlap detected earlier becomes structure instead of a problem.
- Parent `android`: form `modular-base` (= android-native-base).
- Template backlog (gradual): platform targets as features (`web`, `desktop`, `ios`) via `tpl:if` in build files + source-set globs; optional capabilities (`room`, `auth`, `ktor`). A `ui`-layer variant of the KMP modular form would be a third form when wanted.

## Registry v2 (draft)

```json
{
  "schema_version": 2,
  "parents": [
    {
      "key": "kmp",
      "label": "Kotlin Multiplatform",
      "forms": [
        {
          "name": "kmp-native-base",
          "form": "modular-base",
          "repo": "Templetry/kmp-native-base",
          "ref": "main",
          "description": "Multi-module production base — Android + Desktop + iOS, convention plugins"
        },
        {
          "name": "compose-multiplatform-app",
          "form": "single-module",
          "repo": "Templetry/compose-multiplatform-app",
          "ref": "main",
          "description": "Single-module starter — Android, iOS, Desktop and Web/Wasm"
        }
      ]
    },
    {
      "key": "android",
      "label": "Android native",
      "forms": [
        {
          "name": "android-native-base",
          "form": "modular-base",
          "repo": "Templetry/android-native-base",
          "ref": "main",
          "description": "Multi-module production base — convention plugins, Compose"
        }
      ]
    }
  ]
}
```

The manifest spec does not change (v1 features already carry the combinable axis). Future sugar, not now: named *presets* in the manifest activating feature combos.
