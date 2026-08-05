# Answers file specification — v1

Every render writes `.templetry-answers.yml` at the output root (ADR-0005). It is what makes template *updates* possible in the future — and it is public API, versioned.

## Shape

```yaml
schema_version: 1
template:
  name: react-vite-ts
  source: local            # local | "github.com/owner/repo@ref/path" for remote renders
  commit: <sha>            # resolved template commit when known (drift detection anchor)
variables:                 # resolved values, keys sorted
  node_version: "22"
  project_name: Demo Shop
features:                  # resolved states, keys sorted
  analytics: false
  router: true
```

## Rules

- Keys are emitted **sorted** and the file is byte-deterministic: same inputs → same file (NFR4; golden tests depend on it).
- **No timestamp in v1** — determinism beats provenance; a `generated_at` field may arrive later behind an explicit option.
- `source` records where the template came from; `local` for directory renders. Forge URL + commit ref land together with remote fetching (Phase 2).
