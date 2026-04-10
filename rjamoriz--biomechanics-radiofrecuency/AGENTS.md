# Repository-wide Copilot instructions

This repository implements a professional treadmill running biomechanics platform
powered primarily by ESP32 Wi-Fi CSI sensing, with realtime streaming, proxy metric
estimation, optional inferred synthetic motion rendering, and validation workflows.

These instructions apply to all code in the repository.

## Product context

The platform is used by coaches, sport scientists, operators, and performance labs.

Core capabilities include:
- ESP32 CSI ingestion
- realtime running analytics
- athlete, station, and session management
- treadmill protocol handling
- calibration workflows
- session replay
- reporting
- validation against external references
- optional inferred 2D/3D motion representations rendered as synthetic views

## Scientific and domain guardrails

Always preserve scientific honesty.

### Wi-Fi sensing is not a camera
Never describe Wi-Fi outputs as optical footage, real video, or camera-equivalent views.

Allowed terminology:
- estimated metric
- proxy metric
- inferred pose
- inferred body model
- synthetic motion view
- rendered lateral view
- rendered rear view
- rendered front view
- confidence
- validation status
- signal quality
- calibration status

Forbidden claims:
- true front view
- true rear view
- true lateral camera view
- exact joint angles without validation
- exact ground reaction force
- exact plantar pressure
- exact center of pressure
- medical diagnosis
- injury diagnosis
- clinical-grade biomechanics without validation

### Distinguish output classes
Keep these categories separate in code, docs, APIs, and UI:
1. direct signal measurements
2. derived proxy metrics
3. inferred motion outputs

Examples:
- direct signal measurements: CSI packets, RSSI, channel, packet rate
- proxy metrics: cadence, step interval, symmetry proxy, contact-time proxy
- inferred motion outputs: 2D keypoints, 3D skeleton, simplified body model, synthetic renders

### Confidence is required
Every estimated metric and inferred motion output should expose, when applicable:
- confidence
- signal quality context
- calibration status or calibration quality
- validation state

Use validation states consistently:
- unvalidated
- experimental
- station-validated
- externally validated

## Architecture rules

Preserve service boundaries.

### apps/web
Use this for:
- operator and coach UI
- athlete, station, and session screens
- live dashboards
- replay and reports
- synthetic inferred motion visualization

Preferred stack:
- Next.js App Router
- React
- TypeScript
- Tailwind CSS
- shadcn/ui
- Zod
- TanStack Query
- TanStack Table
- Recharts
- react-three-fiber or three.js for synthetic motion views

### apps/gateway
Use this for:
- serial ingestion from ESP32
- normalization and buffering
- rolling feature extraction
- realtime metric estimation
- signal quality and confidence logic
- websocket streaming
- inference adapter integration
- forwarding normalized events to backend

Preferred stack:
- Node.js
- NestJS
- TypeScript

### apps/backend
Use this for:
- source of truth for domain data
- auth and roles
- athlete, station, session, protocol, calibration, validation, reporting domains
- persistence
- historical analytics and summaries

Preferred stack:
- Java 21
- Spring Boot
- Spring Web
- Spring Security
- Spring Data JPA
- Bean Validation
- Flyway
- PostgreSQL

### ml
Use this for:
- preprocessing
- dataset building
- offline training
- evaluation
- model export
- inference contracts

Preferred stack:
- Python 3.11
- PyTorch
- ONNX when useful

### firmware
Use this for:
- ESP32 transmitter/AP node
- ESP32 receiver/CSI collector
- stable serial packet format
- hardware-facing code only

## Implementation standards

When writing or editing code:
- prefer production-style structure over demo-style shortcuts
- keep files cohesive and reasonably small
- use clear names and strong typing
- validate DTOs and API boundaries
- add useful logs
- avoid hidden assumptions
- avoid overengineering
- keep the repo runnable locally
- update tests when behavior changes
- update docs when claims, workflows, or architecture change

Prefer names like:
- estimatedCadence
- stepIntervalEstimate
- symmetryProxy
- contactTimeProxy
- flightTimeProxy
- fatigueDriftScore
- signalQualityScore
- metricConfidence
- inferredPose
- inferredMotionFrame
- syntheticViewType

Avoid names like:
- exactCadence
- trueKneeAngle
- realRearView
- cameraViewFromWifi

## UI and copy rules

The UI must look like a professional sports-tech product, not a hobby prototype.

Required behaviors:
- loading states
- empty states
- error states
- accessible forms
- confidence indicators
- clear warnings for synthetic inferred motion

Whenever synthetic front/rear/lateral motion is displayed, include wording equivalent to:
- "This is a synthetic model-based rendering inferred from Wi-Fi sensing."
- "It is not a true camera or optical motion capture view."

Do not hide uncertainty.

## Data and validation rules

Raw CSI files should be stored as raw captures with metadata.
Do not treat inferred motion outputs as camera footage.

Support validation workflows against imported references such as:
- treadmill console exports
- IMU CSV
- video-derived CSV
- pressure insole CSV
- force plate CSV

Do not silently mix validated and experimental outputs.

## Testing rules

Add or update tests for meaningful changes.

Priorities:
- frontend component/page tests
- gateway parser and websocket tests
- backend controller/service/repository tests
- Testcontainers for PostgreSQL where appropriate
- ML preprocessing and inference contract tests
- warning-banner tests for inferred synthetic motion screens

## Documentation rules

Keep docs aligned with the implementation.

Prioritize updates to:
- docs/architecture.md
- docs/sensing_limitations.md
- docs/inferred_views.md
- docs/calibration_protocol.md
- docs/validation_workflow.md
- docs/hardware_setup.md

Documentation must clearly explain:
- what is directly measured
- what is estimated
- what is inferred
- what is experimental
- what is validated
- what is still unknown

## Developer workflow

When making changes:
1. inspect the relevant code path first
2. preserve architectural boundaries
3. make the smallest clean change that solves the problem
4. update types and schemas
5. update tests
6. update docs if needed

If a requirement is scientifically ambiguous, choose the more conservative interpretation.
If hardware behavior cannot be fully verified, scaffold professionally and document assumptions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rjamoriz)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/rjamoriz)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
