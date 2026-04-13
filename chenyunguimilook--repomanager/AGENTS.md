# RepoManager — AI Coding Assistant Guide

A macOS SwiftUI app for managing multiple Git repositories: batch operations, status monitoring, version tagging, and external tool integration.

## Architecture Overview

**Three-Layer Structure:**
- **UI Layer:** `ContentView.swift` — SwiftUI Table + toolbar + context menus for repo management
- **State Layer:** `RepoListViewModel.swift` — `@MainActor` ObservableObject managing repos array, persistence, concurrent refresh, and import workflow
- **Git Layer:** `GitService.swift` — `nonisolated` static functions wrapping `/usr/bin/git` commands with structured output parsing

**Data Model:** `GitRepo` (in `RepoStatusType.swift`) — `Identifiable, Codable, Sendable` struct. Only `id`, `path`, `name` persist to disk; runtime fields (`branch`, `statusType`, `latestTag`, `currentOperation`) are refreshed on launch.

**Supporting Components:**
- `AppStartManager.swift` — Login item management via `SMAppService`
- `GitRepo+Actions.swift` — File system operations (open in VSCode, clean `.build`)
- `Version.swift` + `Version+Behavior.swift` — Semantic version parsing and increment logic
- `RepoManagerApp.swift` — Window/MenuBar/Settings scene configuration

## Critical Concurrency Patterns

**Actor Isolation:**
- ViewModel is `@MainActor` — all `@Published` property updates must occur on main thread
- `GitService` functions are `nonisolated` — designed for background execution
- When adding async helpers, maintain this boundary: UI state on MainActor, I/O operations nonisolated

**Parallel Execution:**
```swift
// Pattern in refreshAll(): Task.detached + withTaskGroup
let repoUpdates = await Task.detached(priority: .userInitiated) {
    return await withTaskGroup(of: (UUID, GitRepo).self) { group in
        for repo in snapshot {
            group.addTask { (repo.id, await GitService.fetchStatus(for: repo)) }
        }
        return await group.reduce(into: []) { $0.append($1) }
    }
}.value
```

**Operation State:** `GitRepo.currentOperation: String?` prevents UI flicker during concurrent operations. Set before async work, clear after. See `batchOperation(label:action:)` for pattern.

## Persistence & Data Flow

**Storage:** `~/Library/Application Support/GitHubble/repos.json` stores only `[{id, path, name}]`. Runtime status rebuilt via `GitService.fetchStatus()` on app launch (triggered from View's `.task {}`).

**Codable Constraints:** `GitRepo.CodingKeys` explicitly omits transient fields. Adding persistent fields requires:
1. Add to `CodingKeys` enum
2. Update `loadFromDisk()` migration logic if needed
3. Ensure `Sendable` conformance (value types or immutable references)

## Git Integration Details

**Command Execution:** `GitService.runCommand(_ args: [String], at path: String)` returns `(output: String, exitCode: Int32)`. Hardcoded to `/usr/bin/git`; if adding CI support, check for `/usr/local/bin/git` or use `/usr/bin/env git`.

**Status Detection Pattern:**
```swift
// In fetchStatus(for:) — typical sequence:
1. Get remote URL: git remote get-url origin
2. Detect project file: scan for .xcodeproj/Package.swift
3. Latest tag: git describe --tags --abbrev=0
4. Check if tag is at HEAD: compare git rev-parse HEAD vs git rev-list -n 1 <tag>
5. Branch status: git rev-parse --abbrev-ref HEAD (handle "HEAD" = detached)
6. Dirty check: git status --porcelain
7. Ahead/behind: git rev-list --count @{u}..HEAD and HEAD..@{u}
```

**Status Priority:** `RepoStatusType` implements `Comparable` for table sorting: error > detached > diverged > dirty > behind > ahead > clean > loading.

## External Tool Integration

**Pattern (see `GitRepo+Actions.swift`):**
1. Check for CLI tool: `FileManager.default.fileExists(atPath: "/usr/local/bin/code")`
2. Try direct execution
3. Fallback to `/usr/bin/env <tool>`
4. Final fallback: `NSWorkspace.shared.urlForApplication(withBundleIdentifier:)` or `open -a`

**Integrated Tools:** VSCode (`code` CLI or bundle ID), SourceTree (`stree`), Xcode, Terminal, Finder.

## Common Workflows

**Building:** Open `RepoManager.xcodeproj` in Xcode (project file is in repo root, not under `RepoManager/` subdirectory). Command-line builds work but require proper signing for distribution:
```bash
xcodebuild -project RepoManager.xcodeproj -scheme RepoManager -configuration Debug build
xcodebuild test -project RepoManager.xcodeproj -scheme RepoManager -destination 'platform=macOS'
```

**Adding Features:**
- New Git operations → Add `nonisolated static func` to `GitService`
- New batch actions → Follow `batchOperation()` pattern in ViewModel
- New UI columns → Update `Table` in `ContentView`, ensure `KeyPathComparator` compatibility

**Debugging:** `currentOperation` field is your friend. Set it to describe long-running tasks for user visibility.

## Global Hotkey Management

**Architecture:** Three-component system for customizable global hotkeys:
1. **`HotKeyManager`** — Carbon API wrapper, registers/unregisters hotkeys, triggers callbacks
2. **`HotKeySettings`** — Persists keyCode + modifiers to UserDefaults, formats readable display strings
3. **`HotKeyRecorder`** — SwiftUI UI for recording new hotkey combinations

**Key Pattern:** When user changes hotkey in settings:
```swift
// HotKeySettings observer posts notification
private func notifyChange() {
    NotificationCenter.default.post(name: NSNotification.Name("HotKeySettingsDidChange"), object: nil)
}

// AppDelegate listens and reregisters
settingsObserver = NotificationCenter.default.addObserver(
    forName: NSNotification.Name("HotKeySettingsDidChange"),
    object: nil,
    queue: .main
) { [weak self] _ in
    self?.hotKeyManager.reregister(with: self?.hotKeySettings)
}
```

**Default Hotkey:** F5 (keyCode: 96, modifiers: 0). Customizable via Settings → Hotkey tab.

## Key Files Reference
- `GitService.swift` — Git command wrappers and repo scanning
- `RepoListViewModel.swift` — State management, persistence, batch operations
- `RepoStatusType.swift` — Data model (`GitRepo`, `RepoStatusType` enum with sort priority)
- `ContentView.swift` — Main UI, context menus, drag-and-drop import
- `GitRepo+Actions.swift` — External tool integrations with fallback chains
- `Version+Behavior.swift` — Semantic version increment logic for tagging workflow
- `HotKeyManager.swift` — Global hotkey registration using Carbon API
- `HotKeySettings.swift` — Hotkey configuration persistence and display formatting
- `HotKeyRecorder.swift` — SwiftUI hotkey recording UI component
- `AppDelegate.swift` — App lifecycle, hotkey initialization and notification handling 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chenyunguiMilook)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/chenyunguiMilook)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
