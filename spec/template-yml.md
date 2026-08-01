# Especificación del manifest `template.yml`

**Estado:** borrador v1 — NO es especificación cerrada. Se valida en seco contra 3 plantillas dispares (KMP, React, Python/FastAPI) antes de congelar nada. Criterio: si una plantilla necesita hacks, el schema está mal, no la plantilla.

El manifest es **la API pública de Templetry**: el frontend lo lee para pintar formularios, el motor para renderizar, el CI de la plantilla para verificar. Llevará `schema_version` desde v1.

## Borrador actual

```yaml
schema_version: 1
name: react-vite-ts
platform: web          # tags de catálogo; el motor los ignora
framework: react

variables:
  - key: project_name
    label: Nombre del proyecto
    type: string
    pattern: "^[A-Za-z][A-Za-z0-9 ]+$"
  - key: node_version
    type: select
    options: ["20", "22"]
    default: "22"

# El motor deriva casings de cada variable string:
# {project_name.pascal} {project_name.kebab} {project_name.snake}
# {project_name.camel} {project_name.flat}

identity:
  - from: "template-app"        # cadena canónica en contenido y rutas
    to: "{project_name.kebab}"
  - from: "TemplateApp"
    to: "{project_name.pascal}"

features:
  - key: router
    label: React Router
    files: ["src/routes/**"]    # incluidos solo si la feature está activa
    patches:                    # JSON Patch RFC 6902
      - file: package.json
        op: add
        path: /dependencies/react-router-dom
        value: "^7.0.0"

verify:
  image: node:22
  run: npm ci && npm run build
```

## Cuestiones abiertas del schema

- `requires`/`conflicts` entre features — ¿v1 o v1.1? (ver ADR-0007)
- ¿Declarar ejecutables explícitamente o confiar en los permisos del tarball?
- Orden de aplicación del mapa de identidad (cadenas largas primero) — ¿configurable o fijo?
- Lista de exclusión por archivo para el renombrado (falsos positivos en URLs/badges).
- Extensión del vocabulario de patch a YAML/TOML: mismo `op/path/value`, ¿mismas semánticas?

## Casos límite que la spec debe cubrir (del estudio §6)

Colisiones de subcadena y entre casings · CRLF/LF (autor en Windows) · permisos de ejecución (`gradlew`) · detección de binarios · `.gitkeep` · BOM/encodings · rutas profundas y límite de 260 chars en Windows · lockfiles con el nombre del proyecto.
