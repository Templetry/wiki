# 2026-08-14 — Industrial pieces, and two bugs worth keeping

The catalog gained its first **industrial pieces** — building blocks that encode a real specification rather than an author's opinion — on `python/fastapi-users`: `rbac`, `api-keys`, `audit-trail`, `soft-delete` and `verifactu`, each CI-verified alone and composed with the rest.

## What made them possible

Two engine features that did not exist when the session started, both discovered by trying to build the pieces:

- **Piece identity maps** (v1.6.0). A piece's variables reached `tpl:var` directives but not the identity map, so a "piece per object" could not rename `TemplateEntity` to `Product`. Pieces now carry their own `identity:`, resolved together with the form's so longest-first ordering stays global.
- **`_pieces/`** (v1.7.0). Inside a Go module, piece sources broke the template's own `go build ./...` — they reference symbols of the package they are applied *into*. Go skips underscore-prefixed directories, so the engine accepts both roots and ADR-0003 holds again for forms that ship pieces.

## Two bugs the CI caught

**The probabilistic one.** `api-keys` parsed `service_handle_secret` with `split("_")`, but `secrets.token_urlsafe` emits base64url, whose alphabet includes `_`. Roughly half of generated keys silently failed to authenticate. One CI job passed and the next failed on identical code — the tell that it was randomness, not composition. Fixed with `split("_", 2)` plus a regression test that *forces* an underscored secret. Lesson recorded in study VIII: security pieces need adversarial vectors, not happy paths.

**The sequencing one.** The `go` parent's CI was pinned to an engine release that was still uploading when the parent was pushed; `curl -sL` wrote a 404 page, `chmod +x` marked it executable, and the job died later with a bare exit 127. All ten parents now use `curl -fsSL`, so the download fails with its HTTP status instead of masquerading as a missing command.

## Verifactu: the value of refusing

`verifactu` was the test of study VIII's thesis. The piece implements the mechanism — append-only records, the SHA-256 fingerprint chain, `verify_chain` that names any record altered after the fact, the event log, the QR payload — and **refuses** three things on purpose: no hardcoded AEAT URL (the regulation defers it to a technical document; a wrong URL fails silently until an inspection), no submission transport (needs a qualified certificate — exposed as a `Submitter` protocol), and no non-Verifactu mode.

Its first test pins the exact fingerprint string against the worked example published with the specification. If that assertion ever breaks, the records are not compliant — which is the whole reason the test exists.

## Takeaway

A piece is worth shipping when it removes work that is *mechanical and easy to get subtly wrong*, and it is trustworthy when it is explicit about where the mechanism ends and a human decision begins.
