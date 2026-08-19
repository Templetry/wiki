# ADR 0017 — Template taxonomy: three axes, one closed vocabulary

**Status:** ✅ Accepted · **Date:** 2026-08-15

## Context

The manifest has carried `platform` and `framework` — two optional single-valued strings — since schema v1, described as "catalog tags; the engine ignores them". Twenty-two forms later, they are neither consistent nor useful:

- `framework: go` and `framework: rust` name **languages**, not frameworks.
- `platform` mixes unrelated ideas: `backend`, `web`, `cli`, `android`, `multiplatform`, `any`.
- A form is often more than one thing. `python/fastapi-users` is a backend *and* a database schema; `kmp/single-module` is multiplatform *and* android *and* ios. One string cannot say that.
- Neither field reaches the **registry**, so nothing can filter on them. `templetry list` prints 22 lines and the desktop shows one flat sidebar. That is already awkward and gets worse with every form added.

The catalog is about to grow further. Retro-fitting a taxonomy across 22 forms is cheap; across 60 it is a project.

## Decision

**Three independent axes, each a list, replacing the two singular fields.**

```yaml
kinds:      [backend, database]   # what it IS — closed vocabulary
languages:  [python]              # open, kebab-case
frameworks: [fastapi, sqlalchemy] # open, kebab-case
```

### 1. `kinds` is a closed vocabulary

`frontend` · `backend` · `database` · `infra` · `multiplatform` · `android` · `ios` · `desktop` · `cli`

Validated by the engine, and an unknown value is an error naming the allowed set. This is the whole point of a filter axis: two templates meaning the same thing must use the same word, and a typo must not silently invent a tenth category. The engine already validates feature and variable keys for the same reason.

Extending the vocabulary is a deliberate act — a new value ships in a minor and is recorded here.

**A form may legitimately declare no kind.** `kinds` answers *what the generated project is*, and `meta/template` generates a template, not a project — none of the nine words is true of it, and inventing a tenth to hold one entry would weaken the axis rather than complete it.

The consequence is deliberate and worth stating plainly: **an untagged form matches no kind filter**, so `meta/template` is visible in the full listing and absent the moment any kind is selected. That is the right behaviour — someone filtering for `backend` is not looking for the template that creates templates — but it is a trap for any *other* form that forgets its taxonomy. The catalog's validator emits a warning for exactly this case, and the desktop's filter tests assert the rule so it cannot change by accident.

### 2. `languages` and `frameworks` are open

New languages and frameworks arrive constantly; a closed list would be a permanent bottleneck for third-party catalogs. Only the **shape** is enforced (`^[a-z0-9]+(-[a-z0-9]+)*$`), which is what stops `C#`, `csharp` and `c-sharp` from being three things.

They are separate axes because they answer different questions. "Show me every Go template" and "show me every template using Axum" are both real, and one field cannot serve both — which is exactly how `framework: go` happened.

### 3. The registry carries them (v2.2, additive)

Each form entry gains `kinds`, `languages` and `frameworks`, copied from its manifest. Filtering reads the registry, which is already fetched; making it read 22 manifests instead would turn a listing into 22 downloads.

`schema_version` stays `2`: the fields are optional and older registries keep validating.

### 4. Filtering semantics

**OR within an axis, AND across axes.** `--kind backend --kind cli --language go` means "(backend or cli) and go". This is what every faceted filter does, and the alternative surprises people.

### 5. `platform` and `framework` are deprecated, not removed

The manifest is public API under the [compatibility policy](../spec/compatibility.md); removing a field needs a major. They keep parsing, they are ignored by filtering, and the spec marks them deprecated. Templetry's own 22 forms migrate now.

No silent fallback from the old fields to the new ones. `platform: web` could mean `frontend`, and guessing would put wrong labels on templates while looking like it worked.

## Consequences

- `templetry list --kind database --language go` becomes possible, and the desktop can group and filter instead of listing.
- Every form must now say what it is. That is a small tax per template and the reason the catalog stays navigable past fifty.
- One migration, once: 22 manifests and 22 registry entries.
- Naming is English, like every other identifier in the project, so the primary axis reads `database` rather than a localized abbreviation.
- A third-party catalog with no taxonomy still works — its forms simply match no filter, which is honest rather than wrong.
