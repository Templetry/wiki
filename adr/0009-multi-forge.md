# ADR 0009 — Repo creation on any forge (multi-forge)

**Status:** Accepted · **Date:** 2026-08-01

## Context

Requirement FR9: the target repo must be creatable on any open git host (GitHub, GitLab, Gitea/Forgejo, Bitbucket, self-hosted), not only GitHub. The engine is unaffected (directory in → directory out); the affected piece is the orchestrator. Key insight: `git push` is already universal — the only forge-specific parts are creating the repo via API and the post-setup.

## Decision

1. **Minimal adapter interface**: `createRepo(name, visibility) → clone URL`. No more API surface than that.
2. **Capability model**: each adapter declares which post-setup it supports (`topics`, `branch_protection`...); the app degrades gracefully instead of enforcing a lowest common denominator.
3. **Universal fallback — "bring your own remote" (BYOR)**: the user creates an empty repo anywhere by hand and pastes the URL; the app only pushes. Covers 100% of git hosts with zero adapters.
4. **Login ≠ forge credentials**: GitHub OAuth identifies the user in the app; creating repos on other forges uses a per-host token (PAT), stored encrypted.

## Consequences

- MVP: GitHub adapter + BYOR. GitLab and Gitea/Forgejo in Phase 4+ based on real usage (Gitea/Forgejo/Codeberg share one API: one adapter serves all three).
- Controlled risk: adapters must not grow API surface (anti maintenance-pit rule).
