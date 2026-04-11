## Summary

TaskMaster is an Android app (see `README.md`). This file gives AI coding agents concise, repo-specific guidance to get productive quickly.

Key facts (discoverable)
- **Language**: Java (not Kotlin)
- **Architecture**: MVVM (Activities ⇄ ViewModels), Repositories mediate Room (local) and Firebase (remote) synchronization.
- **Build**: Gradle wrapper (./gradlew). Android Studio is the recommended dev environment. `google-services.json` is required in `app/` for Firebase features.
- **6 Screens**: Splash, Login, Register, TaskList, TaskDetail, AddEditTask

Quick start (actions you can run)

- Install Android Studio + Android SDK (API 21+).
- Add Firebase config: copy `google-services.json.template` to `google-services.json` in `app/` and update with your Firebase project credentials.
- Build debug APK: `./gradlew assembleDebug`
- Run unit tests: `./gradlew test`
- Run instrumentation tests: `./gradlew connectedAndroidTest` (requires device/emulator)

Where to look (concrete file patterns)

- **ViewModels**: `app/src/main/java/com/icradle/taskmaster/viewmodel/*ViewModel.java` — UI state and business orchestration (AuthViewModel, TaskViewModel).
- **Repositories**: `app/src/main/java/com/icradle/taskmaster/data/repository/TaskRepository.java` — sync and data merging logic (look for `syncWithFirebase()` methods).
- **Persistence**: `app/src/main/java/com/icradle/taskmaster/data/db/` — Room database, DAOs (expose `LiveData<List<Task>>`).
- **Activities**: `app/src/main/java/com/icradle/taskmaster/ui/**/*Activity.java` — UI controllers that observe ViewModels via LiveData.
- **Layouts**: `app/src/main/res/layout/activity_*.xml` — Material Design 3 XML layouts.

Integration & cross-component patterns

- **Firebase integration**: Auth (`FirebaseAuth.getInstance()`), Realtime DB (`FirebaseDatabase.getInstance()`). Setup in `app/build.gradle` and `google-services.json`.
- **Sync strategy**: `TaskRepository.syncWithFirebase()` reconciles Room and Firebase. Cloud data is preferred on conflicts. Unsynced tasks are queued via `getUnsyncedTasks()`.
- **LiveData observation**: Activities observe `LiveData` from ViewModels using `viewModel.getAllTasks().observe(this, tasks -> {...})` pattern.
- **Room threading**: Database operations run on `TaskDatabase.databaseWriteExecutor` (ExecutorService with 4 threads).

Agent editing rules (what to do and what to avoid)

- Preserve MVVM separation: move or add logic into `ViewModel` or `Repository` classes; keep UI classes (Activities/Fragments) thin.
- Add focused unit tests for repository and ViewModel changes. Prefer JVM tests with in-memory Room for fast feedback; only add instrumentation tests when necessary.
- Update `README.md` and the instructions here when changing public behavior, Firebase schema, or build flags.
- Do not commit `google-services.json` or other secrets. Ask the repo owner for a test file if needed.

Security / scanning

- This repo includes Snyk instructions (see `.github/instructions/snyk_rules.instructions.md`): run static security scans on newly generated code and fix findings before finalizing changes.

What to know about the 6 screens

1. **SplashActivity** - Checks `FirebaseAuth.getCurrentUser()`, navigates to Login or TaskList after 2s delay
2. **LoginActivity** - Email/password form, calls `AuthViewModel.login()`, observes `userLiveData` for success
3. **RegisterActivity** - Email/password/confirm form, calls `AuthViewModel.register()`, validates password length ≥6
4. **TaskListActivity** - RecyclerView with `TaskAdapter`, FAB for add, menu for filter/logout
5. **TaskDetailActivity** - Displays task from `taskViewModel.getTaskById(taskId)`, edit/delete in menu
6. **AddEditTaskActivity** - Form with DatePickerDialog, AutoCompleteTextView for priority, saves via `taskViewModel.insert()/update()`

Common patterns in this Java codebase

- Activities use `findViewById()` for view binding (no ViewBinding enabled by default, but declared in gradle)
- ViewModels obtained via `new ViewModelProvider(this).get(AuthViewModel.class)`
- LiveData observation: `viewModel.getLiveData().observe(this, data -> { /* update UI */ })`
- Firebase user ID retrieved: `FirebaseAuth.getInstance().getCurrentUser().getUid()`

---
Generated from the repository snapshot on disk. Please review and tell me which areas to expand or correct.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IcradleInnovationsLtd)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/IcradleInnovationsLtd)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
