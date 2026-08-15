# Study IX — Verifactu: what a compliant piece must and must not do

**Date:** 2026-08 · **Status:** study — feeds the `verifactu` piece · Follows [study VIII](industrial-pieces-v1.md) tier 2

## Why this one gets its own study

Every other piece in the catalog can be wrong in a way the user notices and fixes. This one cannot: a billing system that *looks* compliant and is not exposes its owner to fines of up to €50,000 per year, and the defect is invisible until an inspection. So the rule for this piece is stricter than for any other: **encode only what is verifiable against the official specification, and refuse to guess the rest.**

## Legal context

Spain's [Real Decreto 1007/2023](https://leyfacturaelectronica.com/reglamento-tecnico-verifactu/) defines the requirements for invoicing software (SIF). Two ways to comply:

- **Verifactu mode** — every invoicing record is sent to the AEAT as it is issued. Lighter obligations in exchange for transparency.
- **Non-Verifactu mode** — records stay local, and the software must then add qualified electronic signatures and stronger self-protection.

Dates already passed: **companies since 2026-01-01, self-employed since 2026-07-01** ([source](https://www.davisa.es/guia-verifactu-2026/)). This is current law, not a future obligation.

## The mechanism

1. **Invoicing records** — each invoice produces an *alta* record; corrections produce *anulación* records. Records are append-only.
2. **Hash chain** — each record carries a SHA-256 fingerprint computed over selected fields **plus the previous record's fingerprint**, so any retroactive edit breaks the chain ([AEAT FAQ](https://sede.agenciatributaria.gob.es/Sede/iva/sistemas-informaticos-facturacion-verifactu/preguntas-frecuentes/huella-hash.html)).
3. **Event log** — the software records its own significant events (start, stop, anomalies, exports).
4. **QR on the invoice** — carrying issuer NIF, series/number, date and total, pointing at an AEAT verification URL.
5. **Retention** — records preserved and legible for the statutory period.

### The fingerprint, exactly

For an *alta* record the fields are concatenated as `name=value` joined by `&`, in this order:

```
IDEmisorFactura, NumSerieFactura, FechaExpedicionFactura, TipoFactura,
CuotaTotal, ImporteTotal, Huella (previous), FechaHoraHusoGenRegistro
```

with spaces removed, trailing decimal zeros irrelevant (`123.1` == `123.10`), absent values rendered as the bare `name=`, and the SHA-256 result in **uppercase hex** ([worked example](https://seoxan.es/articulo/huella-hash-verifactu-calculo-sha256)):

```
IDEmisorFactura=89890001K&NumSerieFactura=12345678/G33&FechaExpedicionFactura=01-01-2024&TipoFactura=F1&CuotaTotal=12.35&ImporteTotal=123.45&Huella=&FechaHoraHusoGenRegistro=2024-01-01T19:20:30+01:00
```

**Provenance caveat, deliberately recorded:** this template comes from secondary sources that reproduce AEAT's *"Detalle de las especificaciones técnicas para la generación de la huella o hash de los registros"*. The piece must therefore make the assembly a single, isolated, documented function with the example above as a test vector, so verifying it against the official PDF is a five-minute job rather than an archaeology exercise.

## What the piece ships — and what it refuses to

**Ships** (mechanism, verifiable, ecosystem-independent):

- Append-only record model with the chain (`huella`, `huella_anterior`), alta and anulación kinds.
- The fingerprint function, isolated and tested against the documented vector, plus chain verification (`verify_chain`) that detects any retroactive edit.
- Event log model for the software's own events.
- QR payload assembly from the four mandatory data items.
- Read-only query endpoints — as with `audit-trail`, **no route can rewrite a record**.

**Refuses to ship, on purpose:**

- **A hardcoded AEAT URL** for the QR or for submission. The regulation itself defers those to a technical document published by the AEAT; a guessed URL that silently fails validation is worse than an obviously missing one. They are piece variables with empty defaults and a documented pointer.
- **The submission transport.** Sending records requires a qualified certificate, the AEAT's SOAP/REST endpoint and its test environment. The piece exposes a `Submitter` interface and a no-op implementation, so wiring the real service is a deliberate act by someone with the certificate in hand.
- **Non-Verifactu mode.** Qualified electronic signature and anti-tamper self-protection are a different (heavier) obligation; claiming to cover them would be dishonest.

This boundary is the whole point: the piece removes the part that is mechanical and easy to get subtly wrong (chaining, immutability, record shape), and leaves visible the part that requires a decision, a certificate and the official document.

## Consequences for the catalog

- **Jurisdictional pieces need a compatibility axis** (study VIII's observation #2): `verifactu` applies to Spain. Until the registry has such a field, the piece says so in its description and README.
- The piece is a strong argument for **piece `requires`**: it needs an invoice entity to hang on.
