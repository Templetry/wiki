# ADR 0004 — Verificación en contenedores declarada en el manifest

**Estado:** Aceptada · **Fecha:** 2026-08-01

## Contexto

Verificar que la salida compila requiere toolchains distintos por ecosistema (Gradle, Node, Cargo...). Instalarlos todos en el host no escala y acopla el motor a los frameworks (contra ADR-0002).

## Decisión

La plantilla declara su verificación en el manifest: `verify: {image, run}` (p. ej. `image: node:22`, `run: npm ci && npm run build`). El motor solo sabe ejecutar un comando en un contenedor Docker.

## Consecuencias

- El host no necesita ningún toolchain; solo Docker.
- En uso interactivo, verify es opcional/asíncrono (compilar Android tarda minutos); en el CI de cada plantilla es obligatorio.
- Encaja con el despliegue previsto (VPS con Docker ya operativo).
