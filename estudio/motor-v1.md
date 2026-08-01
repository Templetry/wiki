# Estudio previo — Motor de generación de repositorios desde plantillas

**Proyecto:** **Templetry** — gestor de arquitecturas, proyectos y frameworks
**Fecha:** agosto 2026 · **Versión:** 1.1 · **Estado:** estudio previo, sin implementación
**Cambios v1.1:** multi-forge — la creación del repo destino debe funcionar en cualquier alojador git abierto, no solo GitHub (RF9, D9).

---

## 1. Objetivo y alcance

Construir el **motor** que genera el contenido de un repositorio nuevo a partir de una plantilla, con estos límites de alcance:

- **Dentro:** renderizado de plantillas multiplataforma (Android, KMP, web, backend, etc.), composición de features, verificación de la salida, CLI local.
- **Fuera (fases posteriores):** web app, OAuth GitHub, creación de repos vía API, catálogo visual. El motor no debe depender de nada de esto.
- **Usuario objetivo:** inicialmente uno (el autor); diseño no cerrado a abrirse a terceros.

**Métrica de éxito del motor:** de comando a proyecto compilable en < 30 segundos (sin contar verify), para cualquier plantilla registrada, sin código específico de framework en el motor.

---

## 2. Requisitos

### Funcionales
| # | Requisito |
|---|---|
| RF1 | Sustituir la identidad de la plantilla (paquetes, nombres, slugs) en contenido **y rutas** de archivos |
| RF2 | Derivar automáticamente variantes de casing de cada variable (Pascal, kebab, snake, flat, camel) |
| RF3 | Inclusión/exclusión condicional de archivos y bloques según features elegidas |
| RF4 | Modificar archivos estructurados sin comentarios (JSON) de forma declarativa |
| RF5 | Copiar binarios intactos, preservando permisos de ejecución |
| RF6 | Dry-run: mostrar el plan de operaciones sin ejecutar nada |
| RF7 | Verificar que la salida compila, sin toolchains instalados en el host |
| RF8 | Registrar en la salida qué plantilla, versión y valores se usaron (archivo de respuestas) |
| RF9 | Crear el repo destino en **cualquier alojador git abierto** (GitHub, GitLab, Gitea/Forgejo, Bitbucket, servidor propio), no solo GitHub. El login de la app puede seguir siendo GitHub OAuth |

### No funcionales
| # | Requisito |
|---|---|
| RNF1 | **Agnóstico:** cero conocimiento de frameworks en el código del motor |
| RNF2 | **La plantilla compila:** todo repo plantilla es un proyecto válido con CI propio |
| RNF3 | Motor = librería pura (directorio entra → directorio sale) + CLI encima; sin HTTP, sin GitHub |
| RNF4 | Determinista: mismos inputs → misma salida byte a byte (base de los golden tests) |
| RNF5 | Mantenible por una persona: el esfuerzo crece con las plantillas, no con el motor |

---

## 3. Estado del arte

| Herramienta | Escuela | Qué aprender de ella | Por qué no adoptarla |
|---|---|---|---|
| **Copier** | Textual (Jinja2, YAML) | Archivo de respuestas + **update con merge a 3 bandas** (única que gestiona el ciclo de vida, no solo el scaffold). Formulario desde YAML de preguntas. | Plantillas con placeholders no compilan; colisión de delimitadores con Gradle/Kotlin; el manifest (API pública) sería de otro |
| **Cookiecutter** | Textual (Jinja2, JSON) | Estándar de facto; hooks pre/post-gen; ecosistema de +4000 plantillas como referencia de necesidades reales | Sin updates ni migraciones; mismos problemas de Jinja |
| **Yeoman / Plop / Hygen** | Programática (JS) | Generadores in-project (añadir un componente a un repo existente) — posible fase futura | Cada plantilla es código a mantener; escala mal para una persona |
| **Nx generators** | Programática (AST) | Árbol de archivos virtual (facilita dry-run y tests); migraciones automatizadas | Acoplado a su ecosistema; complejidad alta |
| **Projen** | Código-primero | Filosofía "la config se regenera, no se edita" — anti-drift | Ata las plantillas a su runtime; modelo mental distinto al buscado |
| **Backstage scaffolder** | Pipeline de acciones | Manifest declarativo que pinta formularios; separación catálogo/ejecución | Peso enterprise; requiere equipo de plataforma |
| **scaffold / stencil / boilr** (Go) | Textual (Go templates) | Binario único sin runtime como formato de distribución del CLI | Conceptualmente son Cookiecutter |
| **GitHub Templates** | Copia plana | La línea base a superar; API `generate` como referencia | Sin variables, sin lógica |

**Conclusión del sondeo:** la cobertura de lenguajes no diferencia (todos los textuales son agnósticos por construcción). Ninguna herramienta combina *plantilla compilable + composición de features + manifest como API*. Ese hueco es la justificación del motor propio.

---

## 4. Espacio de diseño — decisiones

### D1. Escuela de templating — PROPUESTA: híbrida
- **(a) Placeholders textuales** (Jinja): máxima expresividad; la plantilla deja de compilar, pierde CI e IDE, colisiona con `${}` de Gradle/Kotlin.
- **(b) Programática**: máximo poder; cada plantilla es un programa. Descartada por RNF5.
- **(c) Proyecto real + rename estructurado**: robusto y verificable; sin condicionales.
- **Propuesta:** (c) + directivas en comentarios + patches estructurados. Justificación: RNF2 vale más cuantas más plataformas haya — el CI de cada plantilla verifica lo que el autor no puede probar a mano en 15 ecosistemas.

### D2. Dónde vive el conocimiento de framework — PROPUESTA: en el manifest
El motor expone 5 operaciones genéricas; `template.yml` las configura. Añadir soporte a un framework = escribir una plantilla, nunca tocar el motor. (Es la anti-decisión de Yeoman/Backstage.)

### D3. Los tres mecanismos de transformación — PROPUESTA
1. **Renombrado por mapa de identidad** (universal): la plantilla declara sus cadenas canónicas (`com.template.base`, `template-app`, `TemplateApp`) y a qué variable+casing mapean.
2. **Directivas en comentarios** (`tpl:var`, `tpl:if` / `tpl:endif`): tabla de estilos de comentario por extensión (~20 entradas: `// # <!-- --> /* */`). Extensión desconocida → copiar sin escanear (seguro por defecto).
3. **Patches estructurados** para formatos sin comentarios: JSON Patch **RFC 6902** (no inventar formato), extendido a YAML/TOML con el mismo vocabulario de operaciones.
- Válvula de escape documentada: archivos `*.tpl` con templating textual completo (pierden la garantía de compilar; uso excepcional).

### D4. Verificación — PROPUESTA: contenedor declarado en el manifest
`verify: {image, run}`. El motor solo sabe `docker run`. Sin toolchains en el host; CI de cada plantilla = matrix sobre combinaciones de features principales.

### D5. Updates de proyectos generados — PROPUESTA: preparar, no implementar
Escribir `.forge-answers.yml` (plantilla, versión/commit, valores) en cada proyecto desde el día 1 — coste cero, imprescindible retroactivamente. El merge a 3 bandas estilo Copier queda para una fase futura; es la pieza cara (meses).

### D6. Lenguaje del motor — **ABIERTA**
| Criterio | Kotlin | TypeScript | Go |
|---|---|---|---|
| Afinidad con el autor | ★★★ | ★★ | ★ |
| Compartir código con la futura web | vía Ktor backend | ★★★ (mismo stack) | ✗ |
| Distribución del CLI | JVM o GraalVM nativo | requiere Node | ★★★ binario único |
| Librerías (casings, JSON Patch, tar, YAML) | ★★ | ★★★ | ★★ |
| Riesgo/novedad | bajo | bajo | medio (lenguaje nuevo) |

Nota: el motor es librería pura (RNF3), así que el lenguaje no condiciona la arquitectura — condiciona distribución y reuso. Decisión necesaria antes del primer prototipo, no antes del schema.

### D7. Gramática exacta de directivas — **ABIERTA**
Mínimo viable: `tpl:var <clave>` (línea) y `tpl:if <expr>` / `tpl:endif` (bloque). Por decidir: ¿`tpl:else`? ¿expresiones (`and`/`or`/`not`) o solo clave de feature? ¿directiva de línea vs de archivo? Principio: cuantas menos, mejor — cada directiva nueva es API pública.

### D8. Nombre del proyecto — **RESUELTA: Templetry**
Elegido tras verificar ~70 candidatos en 6 tandas (agosto 2026). Derivado de *templet*, la grafía original de "template" (y herramienta real del telar). Verificado: GitHub (`templetry`, org exacta) y npm libres; ningún producto existente con ese nombre; distancia suficiente de Templafy/Templify.
Consecuencias: CLI `templetry` (alias corto `tpl`), archivo de respuestas `.templetry-answers.yml`, prefijo de directivas `tpl:` (coherente con la marca). Finalistas descartados con pleno de disponibilidad, por si hiciera falta plan B: Repojig, Forgehand.

### D9. Multi-forge (RF9) — PROPUESTA: adaptadores mínimos + capacidades + fallback universal
El motor no se ve afectado (RNF3); solo el orquestador. Reparto del problema:
- `git push` es universal — funciona contra cualquier alojador sin adaptación.
- Lo específico por forge son solo **crear el repo vía API** y el **post-setup**.

Diseño propuesto:
1. **Interfaz de adaptador mínima:** `createRepo(nombre, visibilidad) → URL clonable`. No abstraer más API que esa (anti-pozo de mantenimiento).
2. **Modelo de capacidades:** cada adaptador declara qué post-setup soporta (`topics`, `branch_protection`, `secrets`…); la app degrada con elegancia, sin mínimo común denominador.
3. **Fallback universal "bring your own remote":** el usuario crea el repo vacío a mano en su alojador y pega la URL; la app solo hace push. Cubre el 100 % de alojadores git con cero adaptadores — los adaptadores son comodidad, no requisito.
4. **Autenticación separada del login:** GitHub OAuth identifica al usuario en la app; crear repos en otros forges requiere token (PAT) por alojador, guardado cifrado. Gitea y Forgejo comparten API casi idéntica (un adaptador sirve para ambos y para Codeberg).

Orden de implementación: BYOR + adaptador GitHub en el MVP; Gitea/Forgejo y GitLab después, según uso real.

---

## 5. Borrador de manifest v1 (a validar en seco)

```yaml
# template.yml — borrador, NO especificación
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
identity:
  - from: "template-app"
    to: "{project_name.kebab}"
  - from: "TemplateApp"
    to: "{project_name.pascal}"
features:
  - key: router
    label: React Router
    files: ["src/routes/**"]          # incluidos solo si la feature está activa
    patches:
      - file: package.json
        op: add
        path: /dependencies/react-router-dom
        value: "^7.0.0"
verify:
  image: node:22
  run: npm ci && npm run build
```

**Plan de validación del schema (§7):** redactar este mismo esquema para tres plantillas dispares — KMP (identidad = package JVM + rutas profundas), React (JSON everywhere), Python/FastAPI (snake_case, pyproject.toml) — y comprobar que ninguna necesita hacks. Si una lo necesita, el schema está mal, no la plantilla.

---

## 6. Catálogo de casos límite (checklist de diseño)

- **Colisiones de subcadena** en el rename: `template-app` dentro de `template-application` o de una URL de badge del README. → Renombrado con límites de palabra o lista de exclusión por archivo.
- **Colisiones entre casings**: `templateapp` (flat) puede aparecer por accidente en texto normal. → Orden de aplicación: cadenas más largas primero; revisar en dry-run.
- **CRLF vs LF** (el autor desarrolla en Windows): normalizar a LF en la salida + `.gitattributes` en plantillas; los golden tests deben ser insensibles o normalizar.
- **Permisos de ejecución** (`gradlew`, scripts): tar los preserva, zip no siempre; en Windows no existen — declarar ejecutables en el manifest o preservar del tarball de GitHub.
- **Detección de binarios**: heurística (bytes nulos) + lista de extensiones; nunca escanear directivas en binarios.
- **Directorios vacíos**: git no los versiona — las plantillas usan `.gitkeep`; el motor no debe romperlos.
- **BOM y encodings**: asumir UTF-8; detectar BOM y preservarlo o rechazarlo con error claro.
- **Rutas profundas al renombrar paquetes** (`src/main/kotlin/com/template/base/…`): renombrado de directorios segmento a segmento; cuidado con límite de longitud de ruta en Windows (260).
- **Lockfiles** (`package-lock.json`, `gradle.lockfile`): contienen el nombre del proyecto → o se regeneran en verify, o entran en el mapa de identidad.
- **`.git` del tarball**: no existe (ventaja del tarball sobre clone) — el repo nuevo nace limpio.
- **Features que se excluyen mutuamente o se requieren**: ¿`requires`/`conflicts` en el manifest? Candidato a v1.1 — no complicar v1.

---

## 7. Plan de validación (sin código de producción)

1. **Validación en seco del manifest** (D6 no necesaria): escribir los 3 manifests del §5 a mano. Criterio: cero hacks.
2. **Golden tests** (cuando haya prototipo): inputs fijos → salida esperada byte a byte, versionada en el repo del motor.
3. **CI por plantilla**: la plantilla se compila a sí misma en cada push + compila la salida renderizada de las combinaciones de features principales (matrix).
4. **Dry-run como producto**: el plan de operaciones legible es a la vez feature de UX y herramienta de depuración del motor.

---

## 8. Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| El schema del manifest se queda corto y hay que romperlo | Alto (es la API pública) | Validación en seco (§7.1); campo `schema_version` desde v1 |
| El rename por identidad produce falsos positivos | Medio | Casos límite §6; dry-run con diff; golden tests |
| Scope creep hacia Backstage | Alto | Alcance del §1; la web es fase 2; features de equipo/marketplace fuera |
| El update 3-bandas nunca llega y los proyectos generados quedan huérfanos | Medio | `.forge-answers.yml` desde el día 1 deja la puerta abierta |
| Verify lento (compilar Android tarda minutos) | Bajo | Verify opcional/asíncrono en uso interactivo; obligatorio solo en CI |
| N adaptadores de forge devoran el mantenimiento | Medio | Interfaz mínima (solo `createRepo`) + modelo de capacidades + fallback BYOR; adaptadores nuevos solo si hay uso real |

---

## 9. Roadmap propuesto

- **Fase 0 — Estudio** *(este documento)* → cerrar D6 (lenguaje), D7 (gramática), D8 (nombre); validar manifest en seco (3 plantillas).
- **Fase 1 — Motor**: librería + CLI (`render`, `plan`); golden tests; migrar la plantilla Android o KMP existente al formato.
- **Fase 2 — Verify + CI**: contenedores; plantillas con CI matrix; segunda y tercera plantilla reales.
- **Fase 3 — Web MVP**: OAuth GitHub, catálogo desde `registry.json`, formulario dinámico desde manifest, crear repo + push (adaptador GitHub + fallback BYOR). Deploy en el VPS (Docker).
- **Fase 4+ (opcional)**: adaptadores GitLab y Gitea/Forgejo, updates 3-bandas, generadores in-project, plantillas de terceros.

---

## 10. Preguntas abiertas para cerrar la fase 0

1. Lenguaje del motor (D6) — necesaria antes del prototipo.
2. Gramática mínima de directivas (D7) — ¿solo `var` + `if/endif`?
3. ~~Nombre del proyecto (D8)~~ — **resuelta: Templetry** (ver D8).
4. ¿Las features de v1 soportan dependencias entre sí (`requires`) o se pospone a v1.1?
5. ¿El manifest declara ejecutables explícitamente o se confía en el tarball?
