# Instrucciones del panel orquestador

## Contexto del proyecto
- Este workspace contiene un panel UI para orquestación de arquitecturas, tareas y ejecuciones.
- La app está hecha con Vite + React + TypeScript.
- La fuente de datos local está en `public/data/orchestrator-panel.json`.

## Estado actual de la UI
- Navegación principal en una barra superior horizontal.
- Inspector como drawer lateral bajo demanda.
- Sin sidebar permanente ni bottom tray permanente.
- La experiencia debe priorizar claridad y baja carga cognitiva.
- Si una propuesta aumenta la carga visual o el número de decisiones visibles, no incluirla.

## Prioridades de diseño
- Reducir pasos antes que añadir funciones.
- Mostrar solo la acción principal por pantalla cuando sea posible.
- Mantener jerarquía visual simple: título, contexto mínimo, acción principal y detalle opcional.
- Preferir listas, bloques y estados vacíos claros sobre paneles decorativos.
- Evitar lenguaje técnico innecesario en la UI; usar español claro y corto.

## Convenciones de edición
- Hacer cambios pequeños y enfocados.
- Mantener el estilo visual simple y consistente.
- No reintroducir paneles o elementos decorativos que aumenten ruido visual sin aportar uso claro.
- Cuando se cambie la estructura de datos, actualizar también tipos, hooks y páginas que dependan de ella.
- Antes de proponer una mejora, comprobar si puede resolverse simplificando una pantalla existente.
- No mover contenido entre muchas vistas si una sola vista con acciones claras resuelve el caso.
- No tocar archivos que no estén relacionados con el cambio.

## Archivos clave
- `src/app/store/ui-store.ts`: estado global del panel.
- `src/lib/query/hooks.ts`: lectura del JSON local.
- `src/lib/types.ts`: tipos de dominio.
- `src/components/layout/MainLayout.tsx`: shell principal.
- `src/components/layout/Topbar.tsx`: navegación y selector de arquitectura.
- `src/components/layout/InspectorDrawer.tsx`: panel lateral de inspección.
- `src/pages/*.tsx`: vistas principales.
- `src/index.css`: sistema visual.

## Flujo recomendado al cambiar la UI
1. Identificar el objetivo funcional.
2. Reducir primero la complejidad de interacción.
3. Ajustar tipos y datos si es necesario.
4. Validar con build y lint.
5. Si algo no mejora la comprensión, descartarlo.

## Validación
- Ejecutar `npm run build` para comprobar compilación.
- Ejecutar `npm run lint` para revisar calidad del código.
- Si cambias código, la validación es obligatoria antes de cerrar la tarea.
- Si la validación falla, corregir el problema en el archivo afectado antes de seguir.

## Comunicación
- Responder en español.
- Ser directo, concreto y orientado a acción.
- Si faltan datos reales, usar placeholders claros y fáciles de reemplazar.
- Si el usuario pide una opción concreta, ejecutar esa opción sin abrir una discusión extra.
- Si un cambio puede hacerse con menos archivos, elegir esa ruta.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarcValls)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MarcValls)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
