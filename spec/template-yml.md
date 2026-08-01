# `template.yml` manifest specification

**Status:** draft v1 — NOT a frozen spec. It gets dry-run-validated against 3 dissimilar templates (KMP, React, Python/FastAPI) before anything freezes. Criterion: if a template needs hacks, the schema is wrong, not the template.

The manifest is **Templetry's public API**: the frontend reads it to render forms, the engine to render output, the template's CI to verify. It carries `schema_version` from v1.

## Current draft

```yaml
schema_version: 1
name: react-vite-ts
platform: web          # catalog tags; the engine ignores them
framework: react

variables:
  - key: project_name
    label: Project name
    type: string
    pattern: "^[A-Za-z][A-Za-z0-9 ]+$"
  - key: node_version
    type: select
    options: ["20", "22"]
    default: "22"

# The engine derives casings from every string variable:
# {project_name.pascal} {project_name.kebab} {project_name.snake}
# {project_name.camel} {project_name.flat}

identity:
  - from: "template-app"        # canonical string in content and paths
    to: "{project_name.kebab}"
  - from: "TemplateApp"
    to: "{project_name.pascal}"

features:
  - key: router
    label: React Router
    files: ["src/routes/**"]    # included only when the feature is on
    patches:                    # JSON Patch RFC 6902
      - file: package.json
        op: add
        path: /dependencies/react-router-dom
        value: "^7.0.0"

verify:
  image: node:22
  run: npm ci && npm run build
```

## Open schema questions

- `requires`/`conflicts` between features — v1 or v1.1? (see ADR-0007)
- Declare executables explicitly, or trust the tarball's permissions?
- Identity-map application order (longest strings first) — configurable or fixed?
- Per-file exclusion list for renaming (false positives in URLs/badges).
- Extending the patch vocabulary to YAML/TOML: same `op/path/value`, same semantics?

## Edge cases the spec must cover (study §6)

Substring and cross-casing collisions · CRLF/LF (author works on Windows) · execute permissions (`gradlew`) · binary detection · `.gitkeep` · BOM/encodings · deep paths and the 260-char Windows path limit · lockfiles containing the project name.
