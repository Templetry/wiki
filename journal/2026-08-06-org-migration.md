# 2026-08-06 — Migration to the Templetry org, post-migration audit

Everything now lives in the GitHub organization **Templetry**: `engine`, `desktop`, `catalog`, `wiki`, the parents `kmp`, `android`, `meta`, and the `.github` org profile. The pre-org template repos (`android-native-base`, `kmp-native-base`, `compose-multiplatform-app`) were folded into the parents as forms and deleted from GitHub.

Post-migration audit results:

- All local clones point at `Templetry/*` remotes and are in sync with `origin/main`.
- `catalog/registry.json` references only org repos; the engine's `DefaultRegistryURL` and CLI help point at the org.
- No stale references to the old repo names remain outside historical study documents (kept as record).
- Releases intact: engine v0.1.0 → v0.2.2 (all-platform binaries + SHA256SUMS), desktop v0.1.0 → v0.2.0 (portable exe + installer).
- CI green across repos on the last completed runs. Two runs stuck in `queued` today were caused by a GitHub Actions major outage (2026-08-06), not by configuration.
- Fixed along the way: stale "Phase 0/Phase 1" status banners in the org profile and engine READMEs, the desktop README (was Wails boilerplate), ADR-0012 missing from the ADR index, and a tracked `desktop.exe` build artifact.

New: [state-of-the-art.md](../state-of-the-art.md) is now the single up-to-date snapshot of the project.
