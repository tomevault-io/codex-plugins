# Copilot Instructions

## Project Overview

This is a [HACS](https://hacs.xyz/) custom integration for Home Assistant that connects Petkit smart pet devices (feeders, litter boxes, water fountains, purifiers) via the `pypetkitapi` library. It lives entirely under `custom_components/petkit/`.

## Linting

Pre-commit hooks run **black**, **ruff**, and **codespell**:

```bash
pre-commit run --all-files          # run all hooks
pre-commit run black --all-files    # run a single hook
```

CI runs hooks via tox:

```bash
python -m tox -e precommit
```

Validation against Home Assistant standards uses:

- `hassfest` (HA's own validation tool)
- HACS validation action

There are no automated tests in this repo beyond pre-commit lint checks.

## Architecture

### Three Coordinators

All data flows through three `DataUpdateCoordinator` subclasses defined in `coordinator.py`:

| Coordinator                        | Purpose                                                | Polling                                                              |
| ---------------------------------- | ------------------------------------------------------ | -------------------------------------------------------------------- |
| `PetkitDataUpdateCoordinator`      | Fetches all device state via `pypetkitapi`             | 60 s default; 5 s "fast" after MQTT event; 190 s when MQTT connected |
| `PetkitMediaUpdateCoordinator`     | Downloads/manages camera media files to `/media`       | Configurable (default 15 min)                                        |
| `PetkitBluetoothUpdateCoordinator` | Opens/closes BLE relay connections for water fountains | Configurable (default 30 min)                                        |

All three are stored in `entry.runtime_data` (a `PetkitData` dataclass defined in `data.py`).

### Smart Polling

`PetkitDataUpdateCoordinator` supports "smart polling": when an MQTT event arrives (via `PetkitIotMqttListener` in `iot_mqtt.py`), it calls `enable_smart_polling(nb_tic)` to temporarily switch to 5-second intervals for a fixed number of ticks, then reverts to 190 s (MQTT connected) or 60 s (no MQTT).

### Entity Pattern

Every HA platform (sensor, switch, button, etc.) follows the same structure:

1. **Descriptor dataclass** — extends `PetKitDescSensorBase` (from `entity.py`) and the matching HA `*EntityDescription`. Key fields:
   - `value: Callable[[device], Any]` — extracts the state; returning `None` signals the entity is unsupported on that device
   - `ignore_types` / `only_for_types` — per-device-model filtering
   - `force_add` — bypasses value-lambda check for specific device types
2. **`COMMON_ENTITIES`** list + **`{PLATFORM}_MAPPING`** dict — maps device types (`Feeder`, `Litter`, etc.) to their descriptor lists
3. **Entity class** — extends `PetkitEntity` (which extends `CoordinatorEntity`) and the HA platform entity

`PetKitDescSensorBase.is_supported(device)` runs at setup time to filter which entities are registered for each physical device.

### Device Filtering in Descriptors

When writing a new entity descriptor, control device compatibility using:

- `only_for_types=["t6", "t7"]` — allowlist by device type string (lowercase)
- `ignore_types=["feeder_mini"]` — denylist
- `force_add=["d4h"]` — skip value-lambda check and always add
- `value` returning `None` or raising `AttributeError` — entity silently excluded

Device type strings come from `pypetkitapi` constants (e.g., `T6`, `T7`, `D4H`, `DEVICES_LITTER_BOX`).

### Unique IDs

Entity unique IDs follow the pattern:

```
{device_type}_{device_sn}_{entity_key}
```

### Translations

All entity `translation_key` values must have a matching entry in `translations/en.json` (and ideally all other locale files). The key structure mirrors Home Assistant's entity translation schema.

### Camera / Streaming

Camera entities use WebRTC via Agora SDK or go2rtc. Supporting files: `agora_api.py`, `agora_rtm.py`, `agora_sdp.py`, `agora_websocket.py`, `go2rtc_stream.py`, `webrtc_common.py`, `whep_mirror.py`. The WHEP mirror HTTP views are registered once at `async_setup_entry` time.

### Notifications & MQTT

- `PetkitIotMqttListener` (`iot_mqtt.py`) subscribes to Petkit's IoT MQTT broker and triggers smart polling on events.
- `PetkitNotificationManager` (`notifications.py`) manages Home Assistant persistent notifications based on coordinator refreshes.

## Key Conventions

- All files start with `from __future__ import annotations`.
- `TYPE_CHECKING` blocks are used for all coordinator/config-entry type imports to avoid circular imports.
- Logging currently uses a mixed pattern: some modules import `LOGGER` from `.const`, while others define a module-local logger with `logging.getLogger(__name__)`. Follow the existing pattern in the file you are editing, and avoid introducing a second logger style within the same module unless the file is being intentionally refactored.
- Use `pypetkitapi` device type constants (`T6`, `DEVICES_LITTER_BOX`, etc.) rather than raw strings when possible.
- Options config is split into named sections: `BT_SECTION` (`"bluetooth_options"`) and `MEDIA_SECTION` (`"medias_options"`).
- Python 3.14 is targeted (see `pyproject.toml` and CI matrix).

---
> Source: [Jezza34000/homeassistant_petkit](https://github.com/Jezza34000/homeassistant_petkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
