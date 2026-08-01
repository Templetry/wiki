# ADR 0001 — Motor propio en lugar de adoptar Copier/Cookiecutter

**Estado:** Aceptada · **Fecha:** 2026-08-01

## Contexto

Existen motores de scaffolding maduros y agnósticos al lenguaje: Copier (el más completo: config YAML, updates con merge a 3 bandas), Cookiecutter (estándar de facto), Yeoman/Plop/Hygen (programáticos), clones en Go (scaffold, stencil). Todos son de escuela textual (placeholders Jinja/similar) o programática. Ninguno combina: plantilla que compila + composición de features + manifest declarativo como API.

Adoptar Copier daría producto rápido, pero: las plantillas con `{{ }}` no compilan (pierden CI e IDE), los delimitadores colisionan con `${}` de Gradle/Kotlin, y el formato de manifest — la API pública del proyecto — quedaría definido por un tercero.

## Decisión

Construir un motor propio. El núcleo es pequeño (5 operaciones genéricas, ver ADR-0002); todo lo periférico (casings, JSON Patch, tarballs, OAuth, APIs de forge) se resuelve con librerías existentes.

## Consecuencias

- Los diferenciadores del proyecto (ADR-0003) son posibles; con Copier no lo serían.
- Se asume el mantenimiento del motor. Mitigación: mantenerlo deliberadamente pequeño (RNF5 del estudio).
- La feature de updates de Copier (merge 3-bandas) no se replica a corto plazo — ver ADR-0005.
