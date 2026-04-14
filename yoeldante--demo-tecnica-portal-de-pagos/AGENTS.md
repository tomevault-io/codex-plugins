# Instrucciones de Workspace

Usa `AGENTS.md` como fuente principal de reglas globales del proyecto.

## Prioridades

1. Respetar el modelo multi-municipio.
2. No hardcodear credenciales ni branding fuera de la configuracion.
3. Considerar el webhook como unica fuente de verdad para pagos.
4. Apoyarse en `skills/` para tareas recurrentes del dominio.
5. Consultar `openspec/specs` antes de cambios funcionales relevantes.

## Documentacion Base

- `PRD.md`
- `AGENTS.md`
- `docs/README.md`
- `docs/AI_CONTEXT.md`
- `openspec/specs/*/spec.md`

## Regla Operativa

Si un cambio afecta pagos, municipios, seguridad o despliegue, actualizar o crear primero el artefacto correspondiente en `openspec/changes`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YoElDante) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
