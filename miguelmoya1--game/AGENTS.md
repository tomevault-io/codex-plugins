# Copilot Instructions: Game Project (Backend & Frontend)

## Arquitectura General

### Backend (NestJS, CQRS)

- El backend está construido con NestJS y sigue la arquitectura CQRS (Command Query Responsibility Segregation).
- **Principio clave:** Los controladores NUNCA acceden directamente a los repositorios. Toda modificación de estado (crear, editar, borrar) se realiza mediante comandos (Command) y sus handlers (CommandHandler), usando el CommandBus.
- Las queries (lecturas) se gestionan con QueryBus y sus respectivos handlers.
- Los comandos encapsulan la intención de modificar el estado y reciben los datos necesarios (por ejemplo, el usuario autenticado y el id del recurso).
- Los handlers de comandos son responsables de:
  - Verificar permisos (por ejemplo, `user.isAdmin()` o usando el PermissionsService).
  - Llamar al repositorio correspondiente para realizar la acción.
  - Lanzar excepciones si no hay permisos o el recurso no existe.
- Los comandos y handlers deben estar registrados en `commands.module.ts` y exportados en `core/application/commands/index.ts`.
- Los endpoints de los controladores sólo deben instanciar el comando y ejecutarlo con el CommandBus, devolviendo la respuesta adecuada (por ejemplo, `{ success: true }`).
- El backend expone una API REST que es consumida por el frontend.

### Frontend (Angular)

- El frontend está construido con Angular y estructurado en módulos, servicios y componentes reutilizables.
- El panel de administración permite crear, editar y borrar sets, items y places.
- Cada card de administración tiene un botón rojo de borrar, que muestra una confirmación nativa (`window.confirm`) antes de proceder.
- Si el usuario confirma, se llama al método de borrado del servicio correspondiente (`deleteItem`, `deleteSet`, `deletePlace`), que hace un fetch DELETE al endpoint REST.
- Tras borrar, se refresca la lista de resultados para reflejar el cambio.
- El frontend espera respuestas `{ success: true }` tras un borrado exitoso.

## Flujo de Borrado (Delete Flow)

1. **Frontend:**

   - El usuario ve una lista de sets, items y places en el admin.
   - Cada card tiene un botón rojo de borrar, que muestra un `window.confirm` antes de proceder.
   - Si se confirma, se llama al método de borrado del servicio correspondiente, que hace un fetch DELETE al endpoint REST.
   - Tras borrar, se refresca la lista.

2. **Backend:**
   - Los endpoints DELETE de los controladores (`items.controller.ts`, `set.controller.ts`, `places.controller.ts`) NO acceden al repositorio directamente, sino que crean y ejecutan un comando de borrado (por ejemplo, `DeleteItemCommand`) usando el CommandBus.
   - Cada comando tiene su handler (`DeleteItemHandler`, etc.) que:
     - Verifica permisos (`isAdmin()` o usando el PermissionsService).
     - Llama al método `delete` del repositorio correspondiente.
     - Lanza excepciones si no hay permisos o el recurso no existe.
   - Los comandos y handlers están registrados en `commands.module.ts` y exportados en `core/application/commands/index.ts`.

## SOFT DELETE

- Todos los deletes en el backend son soft delete: los recursos tienen un campo `deletedAt` (fecha).
- Para considerar un recurso como "no borrado", en los métodos de búsqueda o listados (find/search/getAll) se debe filtrar por `deletedAt: null` o por fecha superior a la actual (si aplica).
- Nunca mostrar ni operar sobre recursos con `deletedAt` no nulo (o en el pasado).
- El método `delete` de los repositorios debe marcar el recurso con la fecha actual en `deletedAt`, no eliminarlo físicamente.
- Los handlers de borrado deben usar este mecanismo.
- Los listados y búsquedas deben filtrar correctamente.

## Resumen de Cambios y Buenas Prácticas

- Se crearon los comandos y handlers de borrado para sets, items y places.
- Se refactorizaron los endpoints DELETE para usar CommandBus y los nuevos comandos.
- Se limpiaron imports y se corrigieron errores de lint/compilación.
- El frontend ya estaba preparado para consumir los endpoints DELETE correctamente.
- Se verificó que la arquitectura CQRS se respeta en todo el flujo de borrado.
- **Nunca accedas al repositorio desde el controller.**
- **Siempre verifica permisos en los handlers.**
- **Mantén los comandos y handlers registrados y exportados correctamente.**
- **El frontend y backend están desacoplados: la API REST es el único punto de contacto.**

## Ejemplo de flujo completo de borrado (end-to-end)

1. El usuario pulsa el botón de borrar en una card de item/set/place en el admin.
2. El frontend muestra un confirm.
3. Si el usuario acepta, el frontend llama a `/api/items/:id` (o `/api/sets/:id`, `/api/places/:id`) con método DELETE.
4. El controller del backend recibe la petición, crea el comando de borrado y lo ejecuta con el CommandBus.
5. El handler del comando verifica permisos y llama al repositorio para borrar.
6. Si todo va bien, el backend responde `{ success: true }`.
7. El frontend refresca la lista.

## Notas para Copilot y desarrolladores

- Si añades nuevos recursos que requieran crear, editar o borrar, sigue el mismo patrón: Command, Handler, registro en el módulo, export en el index, y uso de CommandBus en el controller.
- Si hay errores de import/export, revisa que los comandos estén correctamente exportados como módulos.
- Si cambias la estructura de permisos, centralízalo en los handlers o en el PermissionsService.
- Si el frontend espera un formato de respuesta concreto, respétalo en el backend.
- Si tienes dudas sobre la arquitectura, revisa este fichero antes de implementar.

---

### ¿Cómo usa Copilot este fichero?

Copilot (o cualquier agente) lee primero este archivo si existe, para adaptar su comportamiento y entender el contexto del proyecto. Así evita errores de arquitectura, sigue las convenciones del equipo y respeta el estilo y los patrones definidos aquí. Si tienes instrucciones especiales para el equipo o para Copilot, añádelas aquí.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miguelmoya1)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/miguelmoya1)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
