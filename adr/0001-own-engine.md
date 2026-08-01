# ADR 0001 — Build our own engine instead of adopting Copier/Cookiecutter

**Status:** Accepted · **Date:** 2026-08-01

## Context

Mature, language-agnostic scaffolding engines exist: Copier (the most complete: YAML config, three-way-merge updates), Cookiecutter (the de facto standard), Yeoman/Plop/Hygen (programmatic), Go clones (scaffold, stencil). All of them belong to the textual school (Jinja-style placeholders) or the programmatic school. None combines: templates that compile + feature composition + a declarative manifest as the API.

Adopting Copier would ship a product fast, but: templates full of `{{ }}` don't compile (losing CI and IDE support), delimiters collide with Gradle/Kotlin `${}`, and the manifest format — this project's public API — would be defined by a third party.

## Decision

Build our own engine. The core is small (5 generic operations, see ADR-0002); everything peripheral (casings, JSON Patch, tarballs, OAuth, forge APIs) is solved with existing libraries.

## Consequences

- The project's differentiators (ADR-0003) become possible; with Copier they wouldn't be.
- We own the engine's maintenance. Mitigation: keep it deliberately small (NFR5 in the study).
- Copier's update feature (three-way merge) is not replicated short-term — see ADR-0005.
