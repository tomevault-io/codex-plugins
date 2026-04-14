# Copilot Instructions

## Project Overview

**fweep** is a front-end web application for creating maps of interactive fiction parser games (like Zork). It is built with TypeScript and React.

## Process documentation

Read migration-process.md for information about how to handle migrations of the map data format, and how to write migration scripts when making breaking changes to the data model.

Read release-process.md for information about how to prepare a release.

## Special commands to the agent

- `@release <version>`: Prepare a release branch for the specified version, following the release checklist and guidelines in release-process.md. This includes drafting release notes, running automated validation, performing manual smoke tests, and opening a PR to merge the release branch back to main when ready.

## Tech Stack

- **Language:** TypeScript
- **Platform:** Web (browser-based, front-end only)

## Coding Conventions

- Use TypeScript.
- Enable and preserve strict TypeScript settings.
- Prefer `const` over `let`; avoid `var`.
- Use named exports over default exports.
- Use descriptive variable and function names.
- Keep functions small and focused on a single responsibility.
- Use async/await over raw Promises where applicable.
- Prefer explicit types for public APIs and core domain models.

## File & Folder Structure

- Keep source code in `frontend/src/`.
- Keep tests in a `frontend/__tests__/` directory.
- Use kebab-case for file names (e.g., `map-editor.tsx`).

## Testing

- Write unit tests for utility functions and core logic.
- Test files should be named `*.test.ts`, `*.spec.ts`, `*.test.tsx`, or `*.spec.tsx` as appropriate.
- Write test-first when adding new features or fixing bugs. 
- Use Test-Driven Development (TDD) principles: write a failing test, implement the feature, then refactor.
- Keep tests in the centralized `frontend/__tests__/` directory unless there is a strong reason to do otherwise.
- Use Jest (with ts-jest, ESM mode) and React Testing Library.
- Use `jest.fn<FunctionSignature>()` (single type parameter with the full function type) instead of the legacy `jest.fn<ReturnType, ArgsType>()` two-parameter form.
- Import `jest` from `@jest/globals` in test files that need `jest.fn`, `jest.spyOn`, etc., since Jest globals are not automatically available in ESM mode.

## Git

- Write clear, concise commit messages in imperative mood (e.g., "Add room linking feature").
- Keep commits focused on a single change.

## Agent Guidance

- When creating new features, start by understanding the existing code structure before making changes.
- Prefer small, incremental changes over large rewrites.
- Always check for and fix TypeScript and React errors after making changes.
- When unsure about a design choice, ask before proceeding.

## Domain Definitions

- **Interactive Fiction:** A genre of games where players interact with the game world through text input, often involving puzzles and exploration.
- **Parser Game:** A type of interactive fiction game where players input text commands to interact with the game world.
- **Parser:** The component that processes user input and translates it into actions within the game world.
- **Map:** A visual representation of the rooms and connections in an interactive fiction game.
- **Map Editor:** The main interface of fweep where users can create and edit their maps.
- **Room:** A location in the interactive fiction game, which can have connections to other rooms.
- **Connection:** A link between two rooms, which can be directional (one-way) or bidirectional (two-way).
- **Direction:** A label for traversing from one room to another, such as "north", "southeast", "up", "in", "out", etc. Some games may have custom directions beyond the standard compass directions. A room maps directions to connections, and multiple directions may map to the same connection.
- **Source Room:** The room from which a one-way connection originates.
- **Target Room:** The room to which a one-way connection leads.
- **Item:** An object that can be placed in a room, which may have properties and interactions.
- **Scenery:** An item that cannot be picked up by the player but can be interacted with or described in the game world. 
- **NPC (Non-Player Character):** A character in the game that is not controlled by the player, which can have interactions and behaviors.
- **Puzzle:** A challenge or obstacle in the game that requires the player to solve it to progress.
- **Inventory:** A collection of items that the player has collected during the game.
- **Container:** An item that can hold other items, such as a box or a chest.
- **Supporter:** An item that can support other items, such as a table or a shelf.
- **Dark Room:** A room that is initially dark and requires a light source to see its contents.
- **Light Source:** An item that can illuminate a dark room, such as a lamp or a torch.

## Major Features

- **Room Creation:** Users can create rooms and specify their properties (name, description, etc.).
- **Connection Creation:** Users can create connections between rooms, specifying one or more room-direction bindings and the connection type (one-way or two-way).
- **Item Management:** Users can add items to rooms, including properties and interactions.
- **Map Visualization:** The application provides a visual representation of the map, showing rooms and connections.
- **Export Functionality:** Users can export their maps in a format compatible with popular interactive fiction parsers (e.g., Inform, TADS).
- **Import Functionality:** Users can import existing maps from supported formats to edit and visualize them in fweep.
- **Local Storage:** Users can save their maps locally in the browser for later editing and visualization.
- **Undo/Redo:** Users can undo and redo their actions while editing the map.
- **Google Drive Integration:** Users can save and load their maps from Google Drive for easy access across devices. This is a stretch goal and may not be implemented in the initial version.

## Map Visualization

- Use a graph-based visualization to represent rooms as nodes and connections as edges.
- Allow users to drag and drop rooms to rearrange the layout of the map.
- Allow users to click on rooms and connections to edit their properties.
- Provide zoom and pan functionality for navigating larger maps.
- Allow users to toggle the visibility of item icons on the map for a cleaner view.
- Allow users to set custom colors for rooms and connections to visually differentiate them.
- Provide a legend or key to explain the meaning of different colors and icons used in the map visualization.
- Implement a minimap feature that provides an overview of the entire map, allowing users to quickly navigate to different sections.
- Allow users to export the map visualization as an image (e.g., PNG) for sharing or documentation purposes.
- Implement a feature to display room descriptions as tooltips when hovering over rooms in the map visualization.
- Allow users to customize the layout of the map visualization, such as choosing between a grid layout or a force-directed layout.
- Implement smart layout algorithms that automatically arrange rooms and connections in a visually readable way while preserving parser-direction semantics where possible. For example, if a room connects north to another room, the layout should prefer placing the target room visually north of the source room, while still resolving conflicts, overlaps, and dense clusters gracefully.
- Provide options for users to customize the appearance of the map visualization, such as changing node shapes, edge styles, and label fonts.




## Architectural Decisions

- UI framework: Use React for building the user interface.
- State management: Use React's built-in state management for local component state, and use Zustand once shared editor state appears.
- Prefer Cytoscape.js for map visualization because it supports multiple layout strategies, including grid and force-directed layouts, and provides a stronger foundation for advanced graph behavior than simpler node-editor libraries.
- Data model for positions: hybrid -- grid by default, manual override when needed.
- Use explicit edge objects in the data model to support advanced features like one-way connections and custom properties on connections.
- Implement persistence using IndexedDB for local storage, and integrate with Google Drive API for optional cloud save/load.
- Persist maps in a JSON format that captures all necessary information about rooms, connections, items, and their properties for easy export and import.
- Use command pattern for implementing undo/redo functionality, where each action is encapsulated as a command object that can be executed and reversed.
- Layout should support both a manual option and an auto-layout option.
- For auto-layout, use a hybrid layout approach: first derive direction-aware preferred positions from the map's connection semantics, then apply a lightweight force-directed or relaxation pass to reduce overlaps, crossings, and spacing problems. The goal is to preserve parser-direction intent when possible rather than treating the map as a generic graph.
- Styling: Use CSS modules or styled-components for styling the application, ensuring that styles are scoped to components and do not leak globally.
- Accessibility: Follow best practices for web accessibility, such as using semantic HTML, providing keyboard navigation, and ensuring sufficient color contrast.
- Testing: Use Jest and React Testing Library for unit and integration tests, focusing on unit and integration tests for core logic and utility functions, and using end-to-end testing with Cypress for critical user flows.


## Persistence and Storage

- Use IndexedDB as the primary local persistence layer for maps, editor state, and autosave data.
- Persist maps in a versioned JSON format so save data can evolve without breaking older files.
- Treat JSON import/export as a first-class feature for backups, sharing, and portability.
- Use Google Drive as an optional cloud save/load integration, not as the only persistence layer.
- Prefer local-first behavior: save locally first, then sync or upload when appropriate.
- Handle save conflicts explicitly; do not silently overwrite newer cloud data.
- Throttle or debounce remote saves to avoid excessive API usage.
- Keep storage logic separate from UI components behind a small data access layer.
- Avoid OPFS unless the app develops a clear need for high-performance file-like local storage.

## Data Model

- Use stable string IDs for all primary entities, including maps, rooms, connections, items, and characters.
- Model rooms and connections as separate top-level collections rather than nesting all connection data inside rooms.
- Represent each connection as an explicit edge object with its own ID, source room ID, target room ID, and metadata. Do not store direction labels on the connection itself.
- Support both one-way and bidirectional connections explicitly; do not assume every connection has an automatic reverse edge.
- Store direction bindings on rooms as a map from direction labels to connection IDs.
- Allow multiple directions in the same room to point to the same connection when that matches the parser game's semantics.
- If a user creates multiple bindings from the same room to the same target room in different directions, prefer reusing the same connection object rather than creating duplicate parallel connections unless the user explicitly indicates otherwise.
- Keep canonical domain data separate from derived UI state such as selection, hover state, temporary drag state, and viewport state.
- Store room positions as coordinates on a shared map canvas, with support for grid-aligned placement and manual override.
- Normalize repeated entities where practical: store references by ID instead of deeply nesting full objects.
- Persist map data in a versioned JSON document with clear top-level sections such as metadata, rooms, connections, items, and view/layout data.
- Keep parser-game semantics in the domain model, including custom directions, dark rooms, containers, supporters, scenery, and NPCs.
- Treat imported data as untrusted input: validate IDs, references, required fields, and schema version before accepting it.
- Prefer pure transformation and validation functions for operations like import, export, migration, normalization, and graph derivation.
- Only one connection binding should exist for a given room and direction; a single direction from one room must not resolve to multiple different connections.
- Treat auto-layout as a pure graph-derivation step that computes suggested room positions from canonical map data without changing connection semantics.

## UI/UX

-  User may edit the map in two modes: a visual interactive mode where they can drag and drop rooms and connections, and a "CLI" mode where they can issue text commands to manipulate the map.
-  In visual mode, users can click on rooms and connections to edit their properties. As much as possible, editing should be done in inputs surrounding the room/connection in the main canvas, rather than in a separate sidebar or modal, to keep the user focused on the map, and to minimize mouse movement. For example, clicking on a room could open an inline form directly below the room for editing its name and description, and clicking on a connection could open an inline form along the edge for editing its properties.
-  Favor keyboard-accessible editing flows alongside mouse interactions; core actions such as selection, moving focus, opening inline editors, confirming edits, and cancelling edits should be available without a mouse.
-  Prefer inline editing near the relevant room or connection over context switching to distant panels, except when the amount of detail clearly requires a dedicated inspector.
-  Keep the main canvas visually readable. Avoid cluttering the map with persistent controls; reveal advanced controls contextually on selection, focus, or hover.
-  Selection state should always be obvious. Users should be able to clearly tell which room, connection, item, or command target is active.
-  Pan, zoom, and selection should feel fast and predictable. Avoid interactions that unexpectedly move the viewport or dismiss the current selection.
-  When actions fail, provide clear, actionable feedback near the source of the problem and in the CLI output when relevant.
-  Prefer progressive disclosure: keep common tasks simple and visible, while advanced properties remain accessible without overwhelming first-time users.
-  Maintain accessible focus management for inline editors, popovers, menus, and dialogs. Opening an editor should move focus appropriately, and closing it should return focus to a sensible place.
-  Support responsive layouts, but prioritize desktop and laptop editing workflows over mobile-first interaction patterns.
-  Use consistent terminology in the UI. Prefer the domain terms defined in this document, such as room, connection, direction, item, and map.
-  The CLI mode is always available as a wide, one- or two-line input at the bottom of the screen. Users enter CLI mode by focusing this input and typing a command. The CLI should provide autocomplete suggestions for commands, room names, item names, and other relevant entities to help users discover available actions and reduce typing.
-  There should be a small ? icon in the corner of the CLI input that users can click to see a quick reference guide of available commands and their syntax.
-  The UI supports light and dark themes, which users can toggle between. The map visualization should adapt to the selected theme for optimal readability.
-  The map UI displays a background grid to help users align rooms. The grid can be toggled on and off for a cleaner view when desired.
-  When dragging to create a new connection, show a live preview of the connection line following the cursor, and highlight potential target rooms when hovering over them to indicate where the connection will be created.

## User Flow

- When a map file is not open, the user sees a selection dialog showing
  - A list of existing maps available in local storage, in reverse chronological order with the most recently edited maps at the top.
  - An option to create a new map.
  - An option to import a map from a file or from Google Drive (if Drive integration is implemented).
- When the user creates a new map, they are taken to the map editor with an empty canvas.
- Do not allow untitled maps to be created; require the user to enter a name for the map as part of the creation flow, and persist it immediately so it appears in the list of available maps.
- Identically named maps will have numeric suffixes appended to ensure unique names in the list of available maps.
- When the user selects an existing map, it is loaded from local storage and displayed in the map editor.
- When the user imports a map, they are prompted to select a file or choose from their Google Drive files. The selected map is then loaded and displayed in the map editor. It is also added to the list of available maps in local storage for future access.
- App uses URL routing to track the currently open map (e.g., `/map/<map-id>`). This allows users to bookmark or share links to specific maps, and ensures that refreshing the page will reload the current map.


## Validation Rules

- Validate imported and loaded map data before it enters the canonical application state.
- Every persisted ID must be unique within its entity type.
- Every room-direction binding must reference an existing connection.
- Every connection must reference existing source and target rooms.
- A single room-direction pair may map to only one connection.
- Multiple directions in the same room may map to the same connection when intended.
- Deleting a room must also remove or repair any connections and bindings that would otherwise become dangling references.
- Imported data with unknown schema versions, missing required fields, or broken references should be rejected or quarantined with explicit user-facing errors.
- Validation should distinguish between blocking errors and non-blocking warnings, such as unreachable rooms or missing optional metadata.
- Prefer pure validation functions that return structured errors and warnings instead of mutating data while validating.

## Code Organization

- Keep front-end code in `frontend/`, in preparation for potential future expansion to include a backend or shared libraries.
- Keep source code under `frontend/src/` and organize it by responsibility rather than by framework concerns alone.
- Prefer separate top-level areas such as `frontend/src/components/`, `frontend/src/domain/`, `frontend/src/state/`, `frontend/src/storage/`, `frontend/src/import-export/`, and `frontend/src/graph/` as the project grows.
- Keep React components focused on rendering and user interaction; avoid embedding core domain rules directly inside UI components.
- Put canonical data types, transformation logic, validation, and map manipulation rules in `frontend/src/domain/`.
- Put shared editor state, selection state, viewport state, and undo/redo integration in `frontend/src/state/`.
- Put IndexedDB, autosave, and Google Drive integration behind storage adapters in `frontend/src/storage/`.
- Keep importers, exporters, schema definitions, and migration logic together in `frontend/src/import-export/`.
- Isolate graph rendering, layout integration, and graph-to-view-model derivation in `frontend/src/graph/`.
- Prefer pure utility modules for reusable calculations such as geometry, direction normalization, and graph derivation.
- Keep tests in the centralized `frontend/__tests__/` directory, organized so domain rules, storage adapters, and import/export behavior remain easy to find.

## Import/Export Rules

- Treat the internal map model as the canonical representation and implement importers/exporters as transformations to and from that model.
- Prefer lossless JSON import/export for the app's native save format whenever possible.
- Keep native save/import round-trippable: exporting and then re-importing a map should preserve IDs, bindings, metadata, and layout data unless explicitly documented otherwise.
- Support schema-version checks and migration during import before data enters the canonical application state.
- Treat external parser formats such as Inform or TADS as adapter formats; do not distort the internal domain model to match any single export target.
- Report import problems with structured, actionable errors that identify the affected entity, field, and reason.
- Distinguish between blocking import errors and non-blocking warnings for partially supported or lossy imports.
- Prefer pure importer/exporter functions that do not directly read from or write to UI state.
- Preserve unknown but safely storable metadata in the native format when feasible, to avoid needless data loss across save/load cycles.
- Add tests for round trips, migrations, invalid inputs, and representative external-format conversions.

## Performance Guidance

- Optimize first for clarity, then for measurable bottlenecks.
- Avoid rerendering the entire map when a single room, connection, or selection state changes.
- Prefer selective subscriptions, memoization, and derived selectors for shared editor state.
- Debounce or throttle expensive work such as autosave, auto-layout, graph recomputation, and remote synchronization.
- Keep drag, pan, and zoom interactions smooth and responsive; avoid heavy synchronous work during pointer or keyboard movement.
- Separate canonical domain updates from expensive derived view-model calculations when practical.
- Design for moderately large maps, but do not complicate the architecture prematurely for extreme scale.
- Measure before introducing complex optimizations, virtualization, or caching layers.
- Prefer incremental updates to graph rendering and layout data over full rebuilds where the rendering library supports them.

## Error Handling Conventions

- Fail loudly in development and gracefully in the user experience.
- Prefer explicit, actionable error messages over silent failures or generic catch-all messages.
- Surface validation, import, export, and save errors near the source of the problem and in a persistent status area when appropriate.
- Never silently discard user edits, imported entities, or storage failures.
- Use structured error objects for domain and storage operations so the UI can render clear recovery guidance.
- Distinguish recoverable user-facing errors from programmer errors; do not hide unexpected programming faults behind generic UI messaging.
- Preserve as much valid user data as possible when part of an operation fails.
- Autosave and cloud-sync failures should be visible to the user, but should not unnecessarily interrupt ongoing editing.
- When an operation cannot be completed, explain what happened, what data was affected, and what the user can do next.

## Non-goals

- Do not assume multiplayer or real-time collaboration.
- Do not require a backend for core editing workflows.
- Do not couple the internal map model to any one export format.
- Do not store derived UI state inside the canonical domain model unless persistence explicitly requires it.
- Do not silently discard invalid imported data; report validation issues clearly.
- Do not make Google Drive or any cloud provider the only supported persistence path.
- Do not optimize prematurely for very large datasets at the cost of clarity in the initial implementation.

## UI mouse operations

- Shift + click on an empty part of the canvas to create a new room at the clicked location.
- Click and drag from a room's directional handle to create a new connection in that direction. Releasing the drag on an empty part of the canvas creates a new target room and completes the connection. Releasing the drag on an existing room creates a connection to that room.

## Connection drawing

 - Connections are drawn starting from the directional handle of the source room, and ending at the directional handle of the target room if it exists, or at the center of the target room if it does not have a directional handle for that connection. The connection line should have an arrowhead pointing towards the target room to indicate directionality. For bidirectional connections, do not draw an arrowhead.
 - Two-way connections should be drawn as a sequence of straight line segments. For example, when connecting Room A to Room B, where the connection leaves Room A heading north and leaves Room B heading east, the connection line should first go straight up from Room A a short distance, and the other end of the line should go straight left from Room B a short distance, and then the line should connect between those two points with a straight segment. The length of the initial straight segments should be consistent for all connections (e.g., 20 pixels) and should be a setting that can be adjusted by the user. 
 - When dragging a room, all connected edges should update in real time to maintain their connections to the room's directional handles or center as appropriate.
 - A one-way connection should have an arrowhead pointing towards the target room, while a two-way connection should not have an arrowhead.
 - A one-way connection should be drawn as a short line extending from the directional handle of the source room just as it does for the initial segment of a two-way connection, and then continue as a straight line towards the center of the target room without an additional segment. This visually distinguishes one-way connections from two-way connections while still maintaining a clear indication of directionality.

 ## Room editing
 - Clicking on a room or connection selects it. Shift-clicking on a room or connection adds it to the current selection. 
 - Double-clicking on a room opens the room editor, which allows editing the room's description and other properties. The room editor should be an overlay on the entire UI. The map should still be visible in the background, but faded, and should not be interactive while the room editor is open. Room edits should be immediately applied to the underlying room model. The room editor should have a close button, or the user can press Escape to close it. Pressing Enter should move the focus to the next input field within the room editor, and should not close the editor. Details of the room editor layout follow:
    - When the room editor is opened, the first input field (the room name) should be focused and its text selected for easy editing.
    - Below the room name, there should be a larger textarea for editing the room description.
    - When the room editor is opened, the map should pan until the edited room is centered horizontally in the viewport, with its vertical position about one third from the top of the viewport.
    - Reuse the room element from the map editor as the name input in the room editor, so that the user can see the room in its normal context while editing its name. 
    - The room editor should be semi-transparent and allow the user to see the map behind it, but should prevent interaction with the map while open to avoid confusion. It should have a narrow drop shadow at the bottom right corner. The general effect should be of a pane of frosted glass floating above the map, with the room being edited clearly visible and editable in its normal context, while the rest of the map is visible but deemphasized in the background. The room editor should not have a visible border or header, to maintain a lightweight feel and keep the user's focus on the room content itself.
   

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rileypb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
