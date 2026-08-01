# ADR 0002 — El motor es agnóstico; el conocimiento vive en el manifest

**Estado:** Aceptada · **Fecha:** 2026-08-01

## Contexto

Templetry debe soportar múltiples plataformas y frameworks (Android, KMP, web, backend...). La tentación natural es escribir adaptadores por ecosistema (plugin Android, plugin React...). Es el camino de Yeoman y Backstage, y es la trampa: cada framework nuevo sería código nuevo en el núcleo, inasumible para una persona.

## Decisión

El motor conoce exactamente **cinco operaciones genéricas**:

1. Copiar archivos (binarios intactos, con permisos)
2. Renombrar por mapa de identidad (contenido y rutas)
3. Filtrar directivas en comentarios
4. Aplicar patches estructurados (JSON/YAML/TOML)
5. Ejecutar un comando de verificación en un contenedor

Todo el conocimiento específico de framework vive en el `template.yml` de cada plantilla. Los campos `platform`/`framework` son tags de catálogo que el motor ignora.

## Consecuencias

- Soportar un framework nuevo = escribir una plantilla, nunca tocar el motor.
- El manifest se convierte en la verdadera API pública del proyecto — es lo más caro de cambiar (versionado con `schema_version` desde v1).
- El esfuerzo del proyecto crece con el número de plantillas, no con el motor (RNF5).
