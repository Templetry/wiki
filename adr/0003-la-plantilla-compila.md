# ADR 0003 — Las plantillas compilan; tres mecanismos de transformación

**Estado:** Aceptada · **Fecha:** 2026-08-01

## Contexto

Escuelas de templating: (a) placeholders textuales (Jinja) — expresivos pero la plantilla deja de ser un proyecto válido: sin CI, sin IDE, y con colisión de delimitadores en Gradle/Kotlin; (b) programática — cada plantilla es un programa a mantener; (c) proyecto real + rename — robusto pero sin condicionales.

Con múltiples plataformas, el principio "la plantilla compila" gana valor: el autor no puede verificar a mano 15 ecosistemas, pero el CI de cada plantilla sí.

## Decisión

Híbrido sobre (c). Cada plantilla es un proyecto real que compila, con identidad canónica conocida (`com.template.base`, `template-app`, `TemplateApp`). Tres mecanismos de transformación:

1. **Renombrado por mapa de identidad** (universal): cadenas canónicas → variable+casing, en contenido y rutas. Casings derivados automáticamente (Pascal, kebab, snake, camel, flat).
2. **Directivas en comentarios** (`tpl:var`, `tpl:if/endif`): sintaxis válida en cualquier lenguaje; tabla de estilos de comentario por extensión (~20 entradas). Extensión desconocida → copiar sin escanear.
3. **Patches estructurados** para formatos sin comentarios (JSON): JSON Patch RFC 6902 declarado en las features del manifest, extendido a YAML/TOML.

Válvula de escape documentada: archivos `*.tpl` con templating textual completo (pierden la garantía de compilar; uso excepcional).

## Consecuencias

- Cada plantilla se compila a sí misma en su CI y compila su salida renderizada (matrix de features).
- Las plantillas se editan con IDE y autocompletado plenos.
- Menos expresividad que Jinja para plantillas muy dinámicas — trade-off asumido.
- Casos límite a cubrir en la spec: colisiones de subcadena, CRLF/LF, permisos, lockfiles, BOM (ver estudio §6).
