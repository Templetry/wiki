# Templetry — Wiki de estudio y diseño

**Templetry** es un gestor de arquitecturas, proyectos y frameworks: una herramienta que genera repositorios listos para trabajar a partir de una biblioteca de plantillas multiplataforma, y los crea en cualquier alojador git (GitHub, GitLab, Gitea/Forgejo, servidor propio) a la velocidad de la luz.

> *Templetry* viene de **templet**, la grafía original de "template" — que además nombraba una herramienta real del telar: la pieza que mantiene la forma de la tela mientras se teje.

## Estado del proyecto

**Fase 0 — Estudio previo.** Sin código todavía, por diseño: primero se cierran las decisiones caras de revertir.

## Los tres principios (resumen del estudio)

1. **El motor no sabe qué es un framework** — todo el conocimiento vive en el `template.yml` de cada plantilla; el motor solo tiene 5 operaciones genéricas.
2. **La plantilla compila** — cada repo plantilla es un proyecto real con CI propio, nunca un esqueleto con placeholders rotos.
3. **Motor primero, web después** — librería pura + CLI; la web app (OAuth GitHub, catálogo, formularios) es una piel encima, en fases posteriores.

## Mapa del repo

| Carpeta | Contenido |
|---|---|
| [`estudio/`](estudio/) | El estudio previo completo (requisitos, estado del arte, riesgos, roadmap) |
| [`adr/`](adr/) | Architecture Decision Records — una decisión por archivo, con estado explícito |
| [`spec/`](spec/) | Especificación en evolución del manifest `template.yml` |
| [`diario/`](diario/) | Bitácora de sesiones de estudio y diseño |

## Cómo se trabaja aquí

Las decisiones se registran como ADRs con estado `abierta → propuesta → aceptada` (o `reemplazada`). Nada se decide "en la cabeza": si no está en una ADR, no está decidido. El [índice de ADRs](adr/README.md) muestra el estado global de un vistazo.

## Roadmap

- **Fase 0 — Estudio** *(actual)*: cerrar decisiones abiertas, validar el manifest en seco contra 3 plantillas dispares.
- **Fase 1 — Motor**: librería + CLI (`templetry render`, `templetry plan`), golden tests.
- **Fase 2 — Verify + CI**: verificación en contenedores, primeras plantillas reales migradas.
- **Fase 3 — Web MVP**: OAuth GitHub, catálogo, formulario dinámico, creación de repos (GitHub + bring-your-own-remote).
- **Fase 4+**: adaptadores GitLab/Gitea, updates con merge a 3 bandas, plantillas de terceros.
