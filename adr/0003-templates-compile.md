# ADR 0003 — Templates compile; three transformation mechanisms

**Status:** Accepted · **Date:** 2026-08-01

## Context

Templating schools: (a) textual placeholders (Jinja) — expressive, but the template stops being a valid project: no CI, no IDE support, and delimiter collisions in Gradle/Kotlin; (b) programmatic — every template is a program to maintain; (c) real project + rename — robust but no conditionals.

With multiple platforms, the "templates compile" principle gains value: the author can't manually verify 15 ecosystems, but each template's own CI can.

## Decision

A hybrid built on (c). Every template is a real, compiling project with a known canonical identity (`com.template.base`, `template-app`, `TemplateApp`). Three transformation mechanisms:

1. **Identity-map renaming** (universal): canonical strings → variable+casing, in content and paths. Casings derived automatically (Pascal, kebab, snake, camel, flat).
2. **Comment directives** (`tpl:var`, `tpl:if/endif`): valid syntax in any language; a per-extension comment-style table (~20 entries). Unknown extension → copy without scanning.
3. **Structured patches** for comment-less formats (JSON): JSON Patch RFC 6902 declared in the manifest's features, extended to YAML/TOML.

Documented escape hatch: `*.tpl` files with full textual templating (they lose the compile guarantee; exceptional use only).

## Consequences

- Every template compiles itself in its CI and compiles its rendered output (feature matrix).
- Templates are edited with full IDE support and autocompletion.
- Less expressive than Jinja for highly dynamic templates — an accepted trade-off.
- Edge cases the spec must cover: substring collisions, CRLF/LF, permissions, lockfiles, BOM (see study §6).
