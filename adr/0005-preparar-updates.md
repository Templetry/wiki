# ADR 0005 — Archivo de respuestas desde el día 1; updates 3-bandas pospuestos

**Estado:** Aceptada · **Fecha:** 2026-08-01

## Contexto

La feature más valiosa del ecosistema (y la más difícil) es la de Copier: actualizar un proyecto ya generado cuando su plantilla mejora, con merge a 3 bandas (plantilla original + proyecto actual + plantilla nueva). Requiere saber qué plantilla, versión y valores se usaron al generar. Sin ese registro, es imposible retroactivamente.

## Decisión

- Cada proyecto generado incluye `.templetry-answers.yml`: plantilla, commit/versión, valores de variables y features. Coste: cero.
- El update con merge a 3 bandas se pospone a Fase 4+ — no condiciona el MVP.

## Consecuencias

- La puerta a updates queda abierta para todos los proyectos generados desde el primer día.
- El formato del archivo de respuestas forma parte de la API pública (versionarlo igual que el manifest).
