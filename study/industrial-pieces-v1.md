# Study VIII — Industrial pieces: what deserves to be a piece

**Date:** 2026-08 · **Status:** study — feeds the piece backlog and a future ADR if the catalog model needs to grow

**Owner's ask:** research pieces that *industrialize* systems — SIPs (identity and role systems), CRUDs of concrete data, company databases and well-known systems.

## The criterion

A piece earns its place when it is **rewritten badly in every project**. Three tests:

1. **Repeated** — every business app grows one (auth, audit, invoicing).
2. **Standardized** — a real specification exists, so the piece encodes a standard instead of one author's opinion. This is the difference between a template and a liability.
3. **Decoupled** — it can be added later without rewriting the app (ADR-0014's rule), typically through a socket.

Anything failing test 2 is where templates hurt most: a hand-rolled permission model or invoice numbering scheme becomes technical debt the moment it meets an auditor.

## Tier 1 — Identity and roles (the SIP)

The single most re-implemented subsystem, and the one with the best standards.

| Piece | Encodes | Why it matters |
|---|---|---|
| **`rbac`** | [ANSI/INCITS 359-2004 (NIST RBAC)](https://dl.acm.org/doi/10.1145/501978.501980): users, roles, permissions, operations, objects, sessions | The reference model. Hand-rolled `is_admin` booleans are the classic dead end; roles + permissions + assignment tables is the vocabulary auditors expect |
| **`scim-provisioning`** | [SCIM 2.0 — RFC 7643/7644](https://www.rfc-editor.org/rfc/rfc7643.html): `/Users`, `/Groups`, `id`/`externalId`/`meta`, PATCH semantics | The protocol Entra ID, Okta and every IdP speak to provision users. Shipping it means a B2B customer can plug their directory in without custom work |
| **`oidc-login`** | OpenID Connect authorization code + PKCE against any provider | Removes password storage entirely for the apps that should not own credentials |
| **`api-keys`** | Hashed key + prefix lookup, scopes, rotation, last-used | Every API grows one; the failure mode (storing keys in plaintext) is severe and common |

Layering matters: `rbac` is the model, `scim-provisioning` is the wire protocol on top of it, `oidc-login` and `api-keys` are alternative front doors. They compose; none of them subsumes another.

## Tier 2 — Compliance and money (jurisdictional, high stakes)

| Piece | Encodes | Why now |
|---|---|---|
| **`verifactu`** | Spain's [Verifactu](https://www.davisa.es/guia-verifactu-2026/) invoicing records: chained hash per invoice, unalterable log, QR, AEAT reporting | **Already mandatory**: companies since 2026-01-01, self-employed since 2026-07-01, with fines up to €50k for non-compliant software. Anyone shipping billing software in Spain needs this *today* |
| **`einvoice-ubl`** | [EN 16931](https://beel.es/facturacion-electronica-obligatoria-2026) semantic model, UBL/Facturae serialization | The interoperable format side of the same obligation (Ley Crea y Crece) |
| **`double-entry-ledger`** | Debit/credit journal with immutable postings | Money in a mutable `balance` column is a bug factory; the ledger pattern is centuries old and unambiguous |

This tier is where a template stops being a convenience and becomes real leverage: the rules are external, non-negotiable and expensive to get wrong.

## Tier 3 — Cross-cutting infrastructure

Repeated in every project, rarely standardized, but with well-known correct shapes:

| Piece | What it adds |
|---|---|
| **`audit-trail`** | Append-only record of who changed what and when — the compliance backbone that [pairs with soft delete](https://byteaether.github.io/2025/building-an-enterprise-data-access-layer-automated-soft-delete/) |
| **`soft-delete`** | `deleted_at` + default filtering, so "delete" preserves history and foreign keys survive |
| **`multi-tenancy`** | `tenant_id` + a request-scoped filter, [composable with the soft-delete filter](https://byteaether.github.io/2025/building-an-enterprise-data-access-layer-automated-soft-delete/) rather than fighting it |
| **`outbox`** | Transactional outbox: events written in the same transaction as the data, published asynchronously — the standard fix for dual-write inconsistency |
| **`rate-limit`** | Token bucket per key/tenant with standard `RateLimit-*` headers |
| **`observability`** | OpenTelemetry traces/metrics wiring, structured logs with request ids |
| **`healthz-readyz`** | Liveness/readiness split with dependency checks — Kubernetes expects both |

## Tier 4 — Domain CRUDs (the `crud-resource` family)

The generic `crud-resource` piece (shipped in `python/fastapi-users` and `go/rest-sqlite`) already covers "one entity, rendered to your name". Domain variants earn their own piece only when the *schema* carries knowledge:

- **`customers`** — parties with tax id, billing/shipping addresses, contact channels (the invoicing pieces consume it).
- **`catalog-products`** — SKU, variants, price with currency and tax class, stock.
- **`orders`** — order/line/status machine, with money as integer minor units.

Rule of thumb: if the piece is only "an entity with name and description", it is `crud-resource --set entity=X`, not a new piece.

## Tier 5 — Integrations with known systems

Adapters to systems everyone uses. Their value is the *correct* wiring, not the API call:

- **`stripe-billing`** — checkout + **webhook signature verification** (the step everyone skips) + idempotency keys.
- **`s3-storage`** — presigned uploads, no proxying bytes through the app.
- **`keycloak`** / **`entra`** — realm/tenant wiring for the `oidc-login` piece.
- **`postgres-migrations`** — versioned migrations where the SQLite forms use idempotent DDL.

## What this implies for the model

Three observations, none of which needs an engine change today:

1. **Pieces will want to depend on pieces** — `scim-provisioning` needs `rbac`; `verifactu` needs an invoice entity. Features already have `requires`/`conflicts` (engine v1.3); pieces should gain the same, expressed against applied pieces in the answers file. **Candidate for the next engine minor.**
2. **Jurisdictional pieces need a compatibility axis** — `verifactu` is Spain-only. A `tags:`/`applies_to:` field would let the UI say so instead of the author documenting it in prose.
3. **Common pieces (study VII phase 2) become urgent here** — `audit-trail` is the same idea in every ecosystem, differing only in implementation. That is the argument for cross-template pieces with per-parent implementations.

## Recommended order

1. **`rbac`** on `python/fastapi-users` — the SIP core, on the form that already owns users. Highest reuse, best standard.
2. **`audit-trail`** + **`soft-delete`** on the same form — small, immediately useful, and they prove the cross-cutting shape.
3. **`api-keys`** — machine access alongside the human one.
4. **`verifactu`** — the highest-value item on this list for anyone billing in Spain, and the one with a real deadline; deserves its own study before implementation (hash chaining and AEAT reporting are not something to improvise).
5. `scim-provisioning`, then Tier 3 and Tier 5 as demand appears.
