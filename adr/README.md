# Architecture Decision Records

Una decisión por archivo. Formato: contexto → decisión → consecuencias, con estado explícito en la cabecera.

Estados: **abierta** (sin decidir) · **propuesta** (candidata, pendiente de validar) · **aceptada** (en vigor) · **reemplazada por ADR-XXXX**.

## Índice

| ADR | Título | Estado |
|---|---|---|
| [0001](0001-motor-propio.md) | Motor propio en lugar de adoptar Copier/Cookiecutter | ✅ Aceptada |
| [0002](0002-conocimiento-en-el-manifest.md) | El motor es agnóstico; el conocimiento vive en el manifest | ✅ Aceptada |
| [0003](0003-la-plantilla-compila.md) | Las plantillas compilan; tres mecanismos de transformación | ✅ Aceptada |
| [0004](0004-verify-en-contenedor.md) | Verificación en contenedores declarada en el manifest | ✅ Aceptada |
| [0005](0005-preparar-updates.md) | Archivo de respuestas desde el día 1; updates 3-bandas pospuestos | ✅ Aceptada |
| [0006](0006-lenguaje-del-motor.md) | Lenguaje de implementación del motor | ⏳ Abierta |
| [0007](0007-gramatica-de-directivas.md) | Gramática mínima de las directivas `tpl:` | ⏳ Abierta |
| [0008](0008-nombre-templetry.md) | Nombre del proyecto: Templetry | ✅ Aceptada |
| [0009](0009-multi-forge.md) | Creación de repos en cualquier forge (multi-forge) | ✅ Aceptada |
