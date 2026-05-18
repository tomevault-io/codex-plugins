## Ha-Tracker — Instrucciones rápidas para agentes de IA

Objetivo: dar a un agente de IA la información mínima y accionable para trabajar en este repositorio.

- Arquitectura general
  - Integración de Home Assistant en `custom_components/ha_tracker` (servidor Python) y una UI estática en `custom_components/ha_tracker/www` (frontend JS/HTML/CSS).
  - Backend expone endpoints REST bajo `/api/ha_tracker/*` (ver `custom_components/ha_tracker/api/*.py`). Ejemplos: `/api/ha_tracker/config`, `/api/ha_tracker/devices`, `/api/ha_tracker/persons`.
  - La integración registra recursos Lovelace y un panel en tiempo de ejecución (ver `__init__.py`): sirve estáticos en `/ha-tracker` y añade panel con `embed_iframe=True`.
  - El manifiesto `manifest.json` contiene la `version` usada para cache-busting de los assets (`?v=`).

- Patrones y convenciones importantes
  - Única instancia: la entrada de configuración es single-instance (se llama `async_set_unique_id(DOMAIN)` en `config_flow.py`). No crear entradas duplicadas.
  - Todo código de integración usa async/await. Evitar operaciones I/O bloqueantes en el hilo del loop; si hace falta usar `hass.async_add_executor_job` o `aiofiles` (ya usado en `_ensure_blueprint` y `get_version_from_manifest`).
  - Registro idempotente de vistas: `register_api_views` en `api/__init__.py` protege con `_VIEWS_REGISTERED`.
  - Validaciones centralizadas: `config_flow.py` contiene `DEFAULTS` y `MINIMUMS` y la función `_validate_minimums` — seguir esos umbrales cuando se modifiquen opciones.
  - Comprobaciones de permisos: varios endpoints devuelven datos solo si `only_admin` es False o el usuario es admin (revisar `DevicesEndpoint` y `PersonsEndpoint`).

- Frontend / desarrollo UI
  - Archivos estáticos en `www/`. `index.html` carga en modo debug `./js/main.js` y en producción `./dist/ha-tracker.js` (detectado por la query `debug` y el parámetro `v`).
  - Para probar cambios sin bundle: abrir `index.html` con `?debug=1` o con `debug` en la query para forzar carga de `./js/main.js`.
  - `globals.js` contiene configuraciones compartidas del frontend importantes: `haUrl` (sobrescribible con `?haUrl=`), `configureConsole()` (controla logs según `enable_debug`), y la lógica de `isActive()` / `updateInterval`.
  - Cache-busting: la integración añade `?v=<version>` a recursos Lovelace y paneles. Actualizar `manifest.json` versión si se quiere forzar rollout.

- Cómo encontrar y cambiar comportamiento del servidor
  - Endpoints y lógica: `custom_components/ha_tracker/api/*.py` (cada endpoint es una clase `HomeAssistantView`). Modificar aquí para cambiar contratos REST.
  - Registro de zonas: `api/zones.py` + llamadas desde `__init__.py` (`register_zones`, `unregister_zones`).
  - Blueprints: plantilla en `blueprints/persons_in_zones_alert.yaml`. El instalador copia el blueprint al path de Home Assistant usando `_ensure_blueprint`.

- Ejemplos concretos útiles
  - Obtener webhook de OwnTracks/GPSLogger/Traccar: funciones `get_owntracks_webhook_url`, `get_gpslogger_webhook_url`, `get_traccar_webhook_url` en `config_flow.py`. Prefieren `cloudhook_url` y si no existe usan `webhook_id` + `get_url(hass, prefer_external=True)`.
  - Endpoint para devices: `/api/ha_tracker/devices` (ver `api/devices.py`) que normaliza batería, velocidad y filtra coordenadas inválidas.
  - Panel y card URLs: `/ha-tracker/assets/ha-tracker-panel.js` y `/ha-tracker/assets/ha-tracker-card.js` (definidas en `__init__.py` como PANEL_URL y CARD_URL).

- Workflows de desarrollador (detectables desde el repo)
  - Para probar la integración en un HA local: copiar el paquete a `custom_components/` del perfil de Home Assistant y reiniciar HA.
  - Para iterar en la UI: abrir `https://<HA_HOST>/ha-tracker/index.html?debug=1&v=<timestamp>` para cargar los archivos `www/js/*.js` sin bundle y forzar cache-bust.
  - Si cambias assets que sirven como recursos Lovelace, actualiza `manifest.json` version o reinicia HA para que `get_version_from_manifest()` actualice el `?v=`.

- Reglas útiles para PRs y cambios automáticos
  - Mantén la compatibilidad async: no introducir I/O bloqueante en funciones `async def` (usa `async_add_executor_job` o `aiofiles`).
  - Cuando toques endpoints, respeta el control `only_admin` y usa `request['hass_user']` para permisos.
  - Si tocas la UI y cambias URLs o módulos, revisa `index.html` (loader) y `globals.js` (haUrl / debug / enable_debug) para evitar romper el arranque.

Si quieres, actualizo o acorto secciones concretas (por ejemplo, añadir comandos de build si me indicas cómo se genera `dist/ha-tracker.js`).

---
> Source: [vgcouso/ha-tracker](https://github.com/vgcouso/ha-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
