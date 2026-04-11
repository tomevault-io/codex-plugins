# Instrucciones del Proyecto para Copilot

Estas instrucciones definen como debe colaborar el asistente en este repositorio.

## Contexto del proyecto

- Nombre: Debian-CustomPostInstall
- Objetivo: automatizar post-instalacion en Debian 13 (Trixie).
- Flujos principales:
  - V1 monolitico: post-install.sh
  - V2 modular: post-install-v2.sh + modulos en modules/v2
- Restriccion critica: no modificar ni reemplazar el kernel.

## Reglas de trabajo obligatorias

1. Seguridad y estabilidad
- Nunca usar comandos destructivos sin aprobacion explicita (ejemplo: reset duro, borrados masivos fuera de alcance).
- No tocar configuraciones de kernel ni bootloader salvo solicitud explicita del usuario.
- Mantener compatibilidad con Debian Trixie y arquitectura detectada.

2. Paridad funcional V1/V2
- Si se agrega una mejora importante en V1, evaluar y proponer equivalente en V2.
- Si se corrige una regresion en V2, evaluar impacto en V1.
- Cuando aplique, documentar diferencias justificadas entre ambos flujos.

3. Estilo de cambios
- Hacer cambios minimos y focalizados.
- Evitar refactors no solicitados.
- Preservar nombres, estructura y estilo existente del proyecto.
- Evitar listas de paquetes hardcodeadas nuevas en scripts: priorizar lectura desde `config/app-library.json` mediante libreria compartida.

4. Validacion antes de cerrar tarea
- Ejecutar validacion de sintaxis Bash en archivos modificados:
  - bash -n post-install.sh
  - bash -n post-install-v2.sh
  - bash -n modules/v2/*.sh (segun cambios)
  - bash -n lib/v2/*.sh (segun cambios)
- Reportar claramente si no se pudo validar algo.

5. Documentacion y trazabilidad
- Actualizar CHANGELOG.md en cada lote de cambios funcionales o de documentacion.
- Si se altera comportamiento operativo, actualizar README.md y docs relevantes.
- Mantener STATUS.md al dia con:
  - En progreso
  - Proximo
  - Riesgos/Bloqueos

## Reglas para respuestas del asistente

- Responder en espanol tecnico y claro.
- Priorizar accion sobre teoria cuando se pidan cambios.
- Incluir resumen breve de:
  - Que se cambio
  - En que archivos
  - Como se valido
- Si hay riesgos, indicarlos de forma explicita y concreta.

## Criterios para propuestas tecnicas

- Preferir soluciones robustas para XFCE, systemd, APT y Flatpak.
- Evitar duplicidad de fuentes de apps cuando genere conflicto de UX.
- Mantener enfoque de bajo riesgo para equipos con recursos limitados.

## Convenciones recomendadas

- Commits (sugerido): Conventional Commits.
- Versionado: SemVer (ejemplo: 8.5.0, 8.5.1).
- Historial: mantener seccion Unreleased en CHANGELOG.md.

## Checklist rapido por tarea

1. Entender requerimiento completo.
2. Localizar archivos afectados.
3. Aplicar cambio minimo.
4. Validar con bash -n.
5. Actualizar CHANGELOG.md y docs si aplica.
6. Actualizar STATUS.md.
7. Entregar resumen final con resultados y pendientes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/correakarl)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/correakarl)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
