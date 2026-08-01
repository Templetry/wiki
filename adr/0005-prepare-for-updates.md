# ADR 0005 — Answers file from day one; three-way-merge updates postponed

**Status:** Accepted · **Date:** 2026-08-01

## Context

The most valuable (and hardest) feature in the ecosystem is Copier's: updating an already-generated project when its template improves, via three-way merge (original template + current project + new template). It requires knowing which template, version, and values were used at generation time. Without that record, it is retroactively impossible.

## Decision

- Every generated project includes `.templetry-answers.yml`: template, commit/version, variable values and features. Cost: zero.
- Three-way-merge updates are postponed to Phase 4+ — they don't gate the MVP.

## Consequences

- The door to updates stays open for every project generated from day one.
- The answers-file format is part of the public API (version it just like the manifest).
