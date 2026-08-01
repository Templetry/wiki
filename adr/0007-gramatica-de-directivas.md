# ADR 0007 — Gramática mínima de las directivas `tpl:`

**Estado:** ⏳ Abierta · **Fecha:** 2026-08-01

## Contexto

Las directivas viven en comentarios (ADR-0003) con prefijo `tpl:` (coherente con la marca Templetry, ADR-0008). Cada directiva nueva es API pública: cuantas menos, mejor.

## Mínimo viable propuesto

- `tpl:var <clave>` — la línea contiene un valor canónico a sustituir (complemento puntual al mapa de identidad).
- `tpl:if <feature>` / `tpl:endif` — bloque condicional por feature.

## Por decidir

- ¿`tpl:else`?
- ¿Expresiones (`and`/`or`/`not`) o solo clave de feature simple?
- ¿Directiva de archivo completo (además de la inclusión por globs del manifest)?
- ¿Las features de v1 soportan `requires`/`conflicts` entre sí, o se pospone a v1.1?

## Decisión

*Pendiente. Se cerrará al validar el manifest en seco contra las 3 plantillas de prueba (estudio §7.1).*
