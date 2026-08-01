# ADR 0006 — Lenguaje de implementación del motor

**Estado:** ⏳ Abierta · **Fecha:** 2026-08-01

## Contexto

El motor es una librería pura + CLI (RNF3), así que el lenguaje no condiciona la arquitectura — condiciona distribución, ecosistema de librerías y reuso con la futura web. Necesaria antes del primer prototipo (Fase 1), no antes.

## Opciones sobre la mesa

| Criterio | Kotlin | TypeScript | Go |
|---|---|---|---|
| Afinidad del autor | ★★★ | ★★ | ★ |
| Compartir código con la web | vía backend Ktor | ★★★ (mismo stack) | ✗ |
| Distribución del CLI | JVM o GraalVM nativo | requiere Node | ★★★ binario único |
| Librerías (casings, JSON Patch, tar, YAML) | ★★ | ★★★ | ★★ |
| Riesgo/novedad | bajo | bajo | medio |

## Decisión

*Pendiente.*
