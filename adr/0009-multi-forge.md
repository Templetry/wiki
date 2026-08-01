# ADR 0009 — Creación de repos en cualquier forge (multi-forge)

**Estado:** Aceptada · **Fecha:** 2026-08-01

## Contexto

Requisito RF9: el repo destino debe poder crearse en cualquier alojador git abierto (GitHub, GitLab, Gitea/Forgejo, Bitbucket, servidor propio), no solo GitHub. El motor no se ve afectado (es directorio-entra-directorio-sale); la pieza afectada es el orquestador. Clave: `git push` ya es universal — lo único específico por forge es crear el repo vía API y el post-setup.

## Decisión

1. **Interfaz de adaptador mínima**: `createRepo(nombre, visibilidad) → URL clonable`. No se abstrae más API que esa.
2. **Modelo de capacidades**: cada adaptador declara qué post-setup soporta (`topics`, `branch_protection`...); la app degrada con elegancia, sin mínimo común denominador.
3. **Fallback universal "bring your own remote" (BYOR)**: el usuario crea un repo vacío a mano donde quiera y pega la URL; la app solo hace push. Cubre el 100 % de alojadores con cero adaptadores.
4. **Login ≠ credenciales de forge**: GitHub OAuth identifica al usuario en la app; crear repos en otros forges usa token (PAT) por alojador, guardado cifrado.

## Consecuencias

- MVP: adaptador GitHub + BYOR. GitLab y Gitea/Forgejo en Fase 4+ según uso real (Gitea/Forgejo/Codeberg comparten API: un adaptador sirve para los tres).
- Riesgo controlado: los adaptadores no pueden crecer en superficie de API (anti-pozo de mantenimiento).
