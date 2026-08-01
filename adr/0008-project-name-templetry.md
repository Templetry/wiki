# ADR 0008 — Project name: Templetry

**Status:** Accepted · **Date:** 2026-08-01

## Context

Criteria: unique when searched, descriptive, short/typeable (it becomes a CLI and a directive prefix), and outside the saturated "-forge" naming space. ~70 candidates were verified across 6 rounds (GitHub and npm availability + existing-product searches). General finding: every real English word and every short pronounceable string is already taken as a GitHub handle.

## Decision

**Templetry** — derived from *templet*, the original spelling of "template" (and a real weaving tool: the piece that keeps the cloth's shape on the loom). Styled like *artistry/tapestry*: "the craft of templets".

Verified (2026-08-01): GitHub org `templetry` free (exact name), npm `templetry` free, no existing product with that name, sufficient distance from Templafy/Templify.

## Consequences

- CLI `templetry`, short alias `tpl`; directive prefix `tpl:`; answers file `.templetry-answers.yml`.
- Reserve the handles as soon as possible (GitHub org + npm placeholder) — availability is not a reservation.
- Documented plan B in case of conflict: **Repojig** and **Forgehand** (both verified fully available with zero collisions).
