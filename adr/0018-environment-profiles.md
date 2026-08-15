# ADR 0018 — Environment profiles: three names, each ecosystem's own mechanism

**Status:** ✅ Accepted · **Date:** 2026-08-16

## Context

Every real project runs in more than one place: a laptop, a staging box, production. Templetry's forms shipped a single configuration — `appsettings.json`, `application.yml`, a hardcoded port — so the first thing anyone did after generating a project was invent an environment scheme. Twenty-six forms, twenty-six inventions.

The temptation is to design one. A `templetry.env.yml`, a shared loader, three files in the same shape everywhere. That would be a mistake for the same reason [ADR-0002](0002-knowledge-lives-in-the-manifest.md) keeps framework knowledge out of the engine: the catalog would be inventing a framework, and every generated project would carry a Templetry-shaped config layer nobody asked for and no ecosystem's tooling understands.

Meanwhile most ecosystems already solved this, and their tooling depends on the solution:

| Ecosystem | Native mechanism |
|---|---|
| ASP.NET | `appsettings.{Environment}.json` + `ASPNETCORE_ENVIRONMENT` |
| Spring Boot | `application-{profile}.yml` + `SPRING_PROFILES_ACTIVE` |
| Vite | `.env.{mode}` + `import.meta.env` |
| Gradle (Android/KMP) | build types and `buildConfigField` |
| Xcode | build configurations and `.xcconfig` |
| Node, Python, Go, Rust | nothing blessed — a convention is needed |

## Decision

**Three profiles, named the same everywhere, implemented the way each ecosystem already does it.**

### 1. The names are `development`, `staging`, `production`

One canonical set across the catalog, so `--feature environments` means the same thing in every form and a person moving between two projects is not relearning vocabulary.

The long spellings win over shorter ones because two ecosystems *mandate* them: ASP.NET's `IsDevelopment()`/`IsProduction()` and Vite's default modes both key on these exact strings. Choosing `dev`/`stg`/`pro` would mean fighting the framework in the two places that already have an opinion, to gain consistency in the places that have none.

### 2. Each form uses its ecosystem's mechanism, not a Templetry one

No shared loader, no common config format, no generated abstraction. A .NET project gets `appsettings.Staging.json`; a Vite app gets `.env.staging`; a Spring service gets `application-staging.yml`. Someone who knows the stack recognises it immediately, and its tooling — IDE completion, framework defaults, deployment platforms — keeps working.

Where the ecosystem has nothing blessed (Node, Python, Go, Rust), the form uses the most conventional option for that stack and reads it through **one typed accessor**, so the rest of the code never reaches for a raw environment variable.

### 3. `environments` is a feature, default on

Additive variation is a feature ([ADR-0011](0011-template-forms.md)). A form without it still compiles and runs; turning it off gives the single-configuration project the catalog shipped before.

### 4. A profile is not three empty files

An implementation counts as done when all three hold:

1. **Three sources exist** and differ in something observable, not just in a name string.
2. **The application reads the active one through a typed accessor**, validated at startup — not scattered `getenv` calls.
3. **A test asserts that loading a named profile yields that profile's values.** Without it, the profiles are decoration that rots on the first rename.

### 5. Committed profiles carry no secrets

The three files hold non-secret defaults: log level, feature flags, a service URL. Real credentials come from the deployment environment or a secret manager, and each form gitignores a local override (`.env.local`, `appsettings.Local.json`) for the values a developer needs on their own machine.

A template that shipped a plausible-looking secret would teach the habit of committing them.

## Consequences

- Twenty-six forms gain a feature that most projects need on day one, in the shape each ecosystem's tooling expects.
- The catalog carries no configuration framework of its own, and never has to maintain one.
- The cost is per-ecosystem work rather than one implementation reused: that is the same trade [ADR-0016](0016-common-pieces.md) accepted for pieces, and it is what keeps generated projects idiomatic.
- Consistency lives in the **names and the guarantees**, not in the file format. Two forms in different languages will not look alike, and should not.
