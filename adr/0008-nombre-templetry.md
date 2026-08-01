# ADR 0008 — Nombre del proyecto: Templetry

**Estado:** Aceptada · **Fecha:** 2026-08-01

## Contexto

Criterios: único al buscarlo, descriptivo, corto/tecleable (será CLI y prefijo de directivas), y sin caer en el saturado espacio "-forge". Se verificaron ~70 candidatos en 6 tandas (disponibilidad en GitHub y npm + búsqueda de productos existentes). Hallazgo general: toda palabra inglesa real y toda cadena corta pronunciable está ocupada como handle de GitHub.

## Decisión

**Templetry** — derivado de *templet*, la grafía original de "template" (y herramienta real del telar que mantiene la forma de la tela). Estilo *artistry/tapestry*: "el oficio de los templets".

Verificado (2026-08-01): org de GitHub `templetry` libre (nombre exacto), npm `templetry` libre, ningún producto existente con ese nombre, distancia suficiente de Templafy/Templify.

## Consecuencias

- CLI `templetry`, alias corto `tpl`; directivas con prefijo `tpl:`; archivo de respuestas `.templetry-answers.yml`.
- Reservar los handles cuanto antes (org GitHub + placeholder npm) — la disponibilidad no es una reserva.
- Plan B documentado por si surgiera un conflicto: **Repojig** y **Forgehand** (ambos verificados con disponibilidad total y cero colisiones).
