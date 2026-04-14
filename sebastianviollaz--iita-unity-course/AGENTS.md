# Curso Unity para Adolescentes — Instrucciones del Workspace

## Contexto del Proyecto

Este workspace contiene un **curso de Unity para estudiantes de 12 a 20 años**.
El curso dura un año dividido en 2 mitades:
- **Primera mitad:** 5-6 proyectos guiados que enseñan Unity de forma progresiva (2-3 clases c/u, 1 clase/semana, ~4 clases/mes)
- **Segunda mitad:** Los alumnos crean su propio juego con lo aprendido

**Unity Version:** 2022 LTS (migración a Unity 6 planificada)

## REGLA FUNDAMENTAL: NO ES UN CURSO DE PROGRAMACIÓN

Los alumnos **NUNCA escriben código**. Todo se configura desde el **Inspector de Unity** usando:
- Arrastrar prefabs a la escena
- Configurar componentes desde el Inspector (`[SerializeField]` fields)
- Conectar scripts mediante **UnityEvents** (drag & drop en el Inspector)

**Si cualquier funcionalidad requiere que un alumno escriba C#, esa funcionalidad VIOLA el modelo del curso.**

## Scripts Compartidos (carpeta Scripts/)

Bloques modulares que los alumnos arrastran a GameObjects y configuran desde el Inspector. Se comunican entre sí mediante UnityEvents y referencias en el Inspector.

### PlayerInputs.cs — Capa 2 (Movimiento)
- **Qué hace:** Captura input del teclado/mouse (WASD, salto, agacharse, correr, cámara, zoom)
- **Campos visibles:** `AxisSettings` (ejes de movimiento), `ActionSettings` (teclas de acción), `CameraSettings` (mouse/cámara)
- **Cómo se usa:** Se agrega a un GameObject vacío. Otros scripts (como `BALLPlayer`) lo referencian en su campo `inputController`
- **NO tiene UnityEvents** — expone campos ocultos (`[HideInInspector]`) que otros scripts leen
- **Métodos invocables desde otros UnityEvents:** `ResetInputs()`

### BALLPlayer.cs — Capa 2 (Movimiento - tipo bola)
- **Qué hace:** Mueve un objeto con física (Rigidbody) tipo bola. Soporta salto, checkpoints, teletransporte
- **Campos visibles (secciones colapsables):** `BallCameraSettings` (cámara), `BallSpeedSettings` (maxSpeed, acceleration), `BallJumpSettings` (groundCheckDistance, jumpForce), `BallEvents` (eventos)
- **Requiere:** Referencia a `PlayerInputs` en el campo `inputController`. Rigidbody se agrega automáticamente con RequireComponent
- **Cámara:** El alumno arrastra una Camera al campo `Player Camera`. NO se auto-asigna. Soporta múltiples jugadores con cámaras distintas
- **UnityEvents:** `OnTeleport`, `OnSaveCheckpoint`, `OnResetToCheckpoint`
- **Métodos invocables desde otros UnityEvents:** `ResetToCheckPoint()`, `SaveCheckPoint()`, `TeleportTo(Transform)`, `SetAcceleration(float)`

### FPSPlayer.cs — Capa 2 (Movimiento - primera persona)
- **Qué hace:** Controlador FPS completo. El alumno camina, corre, salta, se agacha y mira con el mouse. Usa CharacterController (se agrega solo con RequireComponent)
- **Campos visibles (secciones colapsables):** `FPSMovementSettings` (walkSpeed, runSpeed, crouchSpeed), `FPSJumpSettings` (jumpForce, gravity), `FPSCameraSettings` (playerCamera, cameraOffset, maxLookUp/Down), `FPSCrouchSettings` (enableCrouch, smoothCrouch, crouchTransitionSpeed, crouchHeight, standingHeight), `FPSCursorSettings` (lockCursorOnStart), `FPSHeadBobSettings` (enableHeadBob, amount, frecuencias), `FPSLandingSettings` (enableLandingImpact, impactAmount), `FPSPlayerEvents`
- **Requiere:** Solo referencia a `PlayerInputs` en el campo `inputController`. CharacterController se agrega automáticamente. AudioListener se gestiona al configurar la cámara
- **Cámara:** El alumno arrastra una Camera al campo `Player Camera`. El script la hace hija del jugador y la posiciona a la altura de los ojos automáticamente. Soporta múltiples jugadores con cámaras distintas
- **UnityEvents:** `OnJump`, `OnTeleport`, `OnSaveCheckpoint`, `OnResetToCheckpoint`
- **Métodos invocables desde otros UnityEvents:** `ResetToCheckPoint()`, `SaveCheckPoint()`, `TeleportTo(Transform)`, `SetWalkSpeed(float)`, `SetRunSpeed(float)`

### OnTriggerEvent.cs — Capa 3 (Eventos/Triggers)
- **Qué hace:** Detecta cuando un objeto entra/sale de un trigger collider, filtrado por Tag
- **Campos visibles:** Array de `TagEvent` — cada uno tiene: `tag` (string), `OnEnter` (UnityEvent), `OnExit` (UnityEvent). Opción `deactivateOnEnter` para coleccionables
- **Requiere:** Un Collider con `Is Trigger = true` en el mismo GameObject
- **Cómo se conecta:** En `OnEnter`/`OnExit` el alumno arrastra un GameObject y selecciona qué método ejecutar (ej: `CountEvent.AddCount`, `GeneralGameManager.LoadScene`)

### CountEvent.cs — Capa 4 (Objetivos/Contadores)
- **Qué hace:** Cuenta ocurrencias. Al llegar al máximo, dispara un evento
- **Campos visibles:** `Text` (UI Text opcional), `Slider` (UI Slider opcional), `MaxCount` (int, mínimo 1), `OnMaxReached` (UnityEvent)
- **Métodos invocables:** `AddCount()`, `ResetCount()` — se conecta desde `OnTriggerEvent.OnEnter` u otros UnityEvents
- **Ejemplo:** 5 monedas → OnTriggerEvent llama `AddCount()` por cada moneda → al llegar a 5 → OnMaxReached dispara `GeneralGameManager.LoadScene("Victoria")`

### TimeEvent.cs — Capa 5 (Tiempo/Presión)
- **Qué hace:** Temporizador configurable (cuenta atrás o hacia adelante) con UI opcional
- **Campos visibles:** `Duration`, `CountDown`, `AutoStart`, `Loop`, `StartDelay`, UI (Text/Slider), `OnStartEvent`, `OnCompleteEvent`
- **Métodos invocables:** `StartTimer()`, `PauseTimer()`, `ResumeTimer()`, `StopTimer()`, `ResetTimer()`, `RestartTimer()`, `SetDuration(float)`
- **Ejemplo:** Timer de 60s → al terminar → `OnCompleteEvent` llama `GeneralGameManager.LoadScene("GameOver")`

### ExtraPlayerInputs.cs — Capa 6 (Inputs Avanzados)
- **Qué hace:** Define teclas personalizadas con eventos para presionar, soltar y mantener
- **Campos visibles:** Array de `ExtraInput` — cada uno: `Name`, `key` (KeyCode), `OnKeyDown`, `OnKeyUp`, `OnKeyHold`, `updateRate`
- **Ejemplo:** Tecla E → `OnKeyDown` llama `Puerta.SetActive(true)` para abrir una puerta

### GeneralGameManager.cs — Capa 6 (Game Loop)
- **Qué hace:** Gestiona el flujo del juego: cargar escenas, pausar, cursor, salir
- **Métodos invocables:** `LoadScene(string)`, `SetDelay(float)`, `PauseGame()`, `ResumeGame()`, `LockCursor()`, `UnlockCursor()`, `QuitGame()`
- **Ejemplo:** Se pone en un GameObject vacío "GameManager". Otros scripts llaman sus métodos vía UnityEvents

## Cadena de Conexión Típica

```
PlayerInputs (lee teclado)
    → BALLPlayer (mueve al jugador tipo bola) O FPSPlayer (primera persona)
        → Jugador toca un trigger
            → OnTriggerEvent.OnEnter dispara
                → CountEvent.AddCount() suma 1
                    → Al llegar a MaxCount → OnMaxReached dispara
                        → GeneralGameManager.LoadScene("Victoria")

TimeEvent (corre en paralelo)
    → Al terminar → OnCompleteEvent dispara
        → GeneralGameManager.LoadScene("GameOver")
```

## Progresión Pedagógica (6 Capas)

| Capa | Proyecto | Concepto Nuevo | Scripts Nuevos | Scripts Acumulados |
|------|----------|---------------|----------------|-------------------|
| 1 | Proyecto 1 | Editor: prefabs, escena, cámara + intro BALLPlayer | `PlayerInputs` + `BALLPlayer` (intro) | PI, BP |
| 2 | Proyecto 2 | Primera persona, cámara FPS, correr/agacharse | `FPSPlayer` | PI, BP, FPS |
| 3 | Proyecto 3 | Triggers, UnityEvents | `OnTriggerEvent` | PI, BP, FPS, OTE |
| 4 | Proyecto 4 | Contadores, objetivos, UI básica | `CountEvent` | PI, BP, OTE, CE |
| 5 | Proyecto 5 | Tiempo, presión, game feel | `TimeEvent` | PI, BP, OTE, CE, TE |
| 6 | Proyecto 6 | Game loop completo (capstone) | `ExtraPlayerInputs` + `GeneralGameManager` | Todos |

### Reglas de progresión:
- Cada proyecto introduce **máximo 1-2 scripts nuevos**
- Los scripts son **acumulativos**: una vez introducido, se puede usar en todos los siguientes
- **Clase 1** = diseño de escena (arrastrar prefabs, posicionar, decorar)
- **Clase 2** = funcionalidad (agregar scripts, configurar Inspector, conectar UnityEvents)
- **Clase 3** (opcional) = mejora/extensión para avanzados, recuperación para rezagados
- Los alumnos avanzados pueden adelantar scripts de la capa siguiente como extensión

## Estructura de Cada Proyecto

```
Proyecto N (NOMBRE)/
├── Scenes/
│   └── Nivel 1.unity
├── Prefabs/           ← Assets específicos (subcarpetas por categoría)
├── Sounds/            ← Audio específico
└── ALCANCE.md         ← Objetivo, pasos por clase, extensiones, errores comunes
```

## Convenciones

- **Nombres de proyecto:** `Proyecto N (TEMA)` — ej: `Proyecto 2 (CARRERA)`
- **Nombres de escena:** `Nivel 1`, `Nivel 2`, etc.
- **Scripts compartidos** van SIEMPRE en `Scripts/`, nunca dentro de un proyecto
- **Prefabs y sounds** van dentro de la carpeta del proyecto específico
- **Render Pipeline:** Built-in (salvo que se indique lo contrario)

## Prompts Disponibles

| Prompt | Cuándo usarlo |
|--------|--------------|
| `crear-proyecto` | Diseñar un proyecto nuevo desde cero |
| `definir-alcance` | Detallar paso a paso las clases de un proyecto |
| `crear-script` | Crear un nuevo bloque modular para los alumnos |
| `mejorar-proyecto` | Diagnosticar y mejorar un proyecto existente |
| `revisar-progresion` | Auditar que el curso completo sea coherente |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SebastianViollaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
