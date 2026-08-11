# `template.yml` manifest specification — v1

**Status:** normative for engine v0.x. The manifest is **Templetry's public API** (ADR-0002): the engine validates against it, the future web reads it to render forms, the template's CI consumes it. Versioned via `schema_version`.

## Full shape

```yaml
schema_version: 1            # required, must be 1
name: react-vite-ts          # required, kebab-case
description: React + Vite + TS starter
platform: web                # catalog tag; the engine ignores it
framework: react             # catalog tag; the engine ignores it

variables:
  - key: project_name        # required, ^[a-z][a-z0-9_]*$, unique
    label: Project name      # optional, for forms
    type: string             # string (default) | select | boolean
    pattern: "^[A-Za-z][A-Za-z0-9 ]*$"   # optional, string type only
    default: ""              # optional; required-ness = has no default
  - key: node_version
    type: select
    options: ["20", "22"]    # required for select
    default: "22"

identity:
  - from: "template-app"     # canonical string, replaced in content AND paths
    to: "{project_name.kebab}"
  - from: "TemplateApp"
    to: "{project_name.pascal}"

features:
  - key: router              # same key rules as variables
    label: React Router
    default: false           # state when the user doesn't choose
    files: ["src/routes/**"] # globs (doublestar); files exist in the template
    patches:                 # RFC 6902 ops; .json, .yml/.yaml and .toml targets
      - file: package.json
        op: add              # add | replace | remove
        path: /dependencies/react-router-dom
        value: "^7.0.0"
  - key: analytics
    requires: [router]       # active analytics demands active router (v1.3)
    conflicts: []            # features that cannot be on together

presets:                     # named feature combos (v1.3)
  - key: full
    label: Everything on
    features: {router: true, analytics: true}

verify:
  image: node:22             # both fields required if verify present
  run: npm ci && npm run build
```

## Semantics

### Variables and casings
Every `string` variable automatically yields casing variants, usable anywhere placeholders are expanded: `{key}` (raw), `{key.pascal}`, `{key.camel}`, `{key.kebab}`, `{key.snake}`, `{key.flat}`. Word splitting handles spaces, hyphens, underscores and camel boundaries (`Mi Super App` → `MiSuperApp`, `mi-super-app`, `mi_super_app`, `miSuperApp`, `misuperapp`).

### Identity map
- Applied to **file content and file paths**, longest `from` first (deterministic, avoids substring shadowing).
- `to` supports placeholders. Dotted identities also apply their **slash form** to both paths and content: `com.template.base` renames `com/template/base/` directories *and* rewrites path references inside docs and scripts.
- Two files mapping to the same destination path is an error.

### Features
A file is **excluded** when it matches the `files` globs of at least one *inactive* feature and matches no *active* feature's globs. Files matching no feature globs are always included. Patches of active features are applied to the (identity-renamed) content of their target file; patch `value` strings expand placeholders.

Since v1.3, patches target **`.json`, `.yml`/`.yaml` and `.toml`** files — the codec is chosen by extension; YAML/TOML go through a JSON round-trip and are re-serialized deterministically (sorted keys, fixed indent). Patches are surgical *data* edits: the patched output file loses comments and hand formatting, so when the format supports comments, prefer directives; reserve patches for JSON and for machine-managed data files.

`requires`/`conflicts` (v1.3): an *active* feature's `requires` must all be active and none of its `conflicts` may be — violations are **errors** at resolution time, never silent auto-enabling. Both lists reference declared feature keys (validated at load).

### Presets
`presets` (v1.3) are named feature combos. Resolution order is **defaults → preset → explicit user choices** — the more specific always wins. An unknown preset name is an error.

### Directives
See [directives.md](directives.md) — normative grammar, comment-style table, and errors.

### Answers file
Every render writes `.templetry-answers.yml` into the output. See [answers-file.md](answers-file.md).

## Validation rules (engine `manifest.Validate`)

- `schema_version` present and equal to 1; `name` present, kebab-case.
- Variable/feature keys match `^[a-z][a-z0-9_]*$` and are unique across their list.
- `select` requires non-empty `options`; `default`, if present, must be one of them.
- `pattern` must compile (RE2).
- Every `identity.to` must expand using only declared variable keys and known casings.
- Patch `op` ∈ {add, replace, remove}; `file` and `path` non-empty.
- `requires`/`conflicts` reference declared feature keys; no self-reference.
- Preset keys match the key regex and are unique; preset features reference declared keys.
- `verify` requires both `image` and `run`.

## Input resolution (render time)

1. Every variable gets a value: user input, else `default`, else **error**.
2. String inputs must match `pattern`; select inputs must be in `options`.
3. Unknown variable, feature or preset names in the inputs are errors (typos never pass silently).
4. Feature states resolve as defaults → preset (if given) → explicit choices; then every active feature's `requires`/`conflicts` are enforced (violations are errors).
