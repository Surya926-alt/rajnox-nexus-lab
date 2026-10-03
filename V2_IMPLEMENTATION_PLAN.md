# NEXUS LAB V2 — V1 Audit and Implementation Plan

## Audit baseline

- **Project:** Flutter app `nexus_lab`, Android-first, extracted from `NEXUS_LAB_first_version.zip`.
- **Application footprint:** one Dart source file, `lib/main.dart` (1,093 lines), plus one smoke widget test.
- **Current dependency reality:** `shared_preferences ^2.5.3` and `webview_flutter ^4.13.0` are already declared. The WebView implementation is real and must be preserved rather than adding a second WebView framework.
- **Current toolchain result:** `flutter` is not installed or discoverable in this sandbox (`flutter: command not found`); `java` is present but `adb` is not. `flutter analyze`, `flutter test`, and the Android build cannot run here until Flutter/Android tooling is available.

## What V1 currently does

`lib/main.dart` contains:

- `NexusLabApp`: Material 3 dark shell with graphite/violet theme.
- `ProjectFile` and `NexusProject`: minimal mutable in-memory file/project models.
- `ExecutionProvider`, `UnavailableExecutionProvider`, `CommandResult`: useful honest execution seam, but not stream/cancellation based.
- `AIProvider`: useful provider seam; currently unused by UI.
- `_NexusShellState`: all navigation, project state, editor, terminal, console, settings, file creation, palette, and panels in one state class.
- `PreviewFrame`: real `webview_flutter` controller and JavaScript channel, but it injects only hard-coded `index.html`, `style.css`, and `script.js`, reloads on content comparison/update, fabricates a fallback entry page, and has no load/error state.

## Preserve

1. **Brand and visual language:** `_bg`, `_panel`, `_panel2`, `_line`, `_violet`, `_cyan`, dark graphite/violet styling, Material 3, responsive wide/phone intent.
2. **Real WebView dependency and controller family:** keep `webview_flutter`; refactor `PreviewFrame` around a stable controller and a proper bundle resolver.
3. **ExecutionProvider and AIProvider concepts:** move them into dedicated files and extend without hard-coding a provider.
4. **Honest unavailable states:** terminal and system metrics should remain unavailable unless a real provider exists.
5. **Existing workspace workflow:** code/preview/console tabs, file list, terminal, projects, tools, monitor, settings, command palette, and Android project scaffolding.
6. **Android application identity:** preserve `com.nexuslab.nexus_lab` unless a V2 product requirement explicitly changes it; correct label/theme/permissions without claiming release signing.

## Refactor

### `lib/main.dart`

Replace the monolithic `_NexusShellState` gradually with an app bootstrap and feature composition. Keep public compatibility for `NexusLabApp`, `NexusShell`, `ProjectFile`, `NexusProject`, `ExecutionProvider`, and `AIProvider` only where tests or downstream code need it; otherwise re-export/import the new model/service files.

### Project model

Replace mutable minimal models with serializable models:

- `Project`: id, name, description, timestamps, files, template type, entry file, `AppConfig`, build history.
- `ProjectFile`: stable id, path, name/extension helpers, content, updatedAt.
- `AppConfig`: app name, package id, icon/splash references, orientation, fullscreen/status/navigation bar, permissions.
- `BuildRecord`: status, target, timestamps, output path/size, logs.

Use immutable/copy-with style where practical so persistence and UI updates are explicit.

### Persistence

Create `ProjectStorage` as a JSON-backed `SharedPreferences` adapter and `ProjectRepository` as the domain API. The repository owns create/load/save/delete/rename/duplicate/list/import/export and active-project selection. Store a structured project bundle per project, never one giant source string. Namespace keys for migration. Duplicate and create operations must guard against accidental overwrite.

Create `SettingsService` over `SharedPreferences` for onboarding completion, font size, auto-save, auto-run, theme/preview settings, and active project id.

### Navigation/state

Create one `AppDestination` enum/source of truth for Workspace, Terminal, Files, Projects, Tools, Monitor, Settings. Map phone destinations explicitly to five safe indices; wide rail can access secondary destinations without indexing a phone `NavigationBar` out of range.

Split feature widgets/services into `lib/features/` and `lib/widgets/` only after models/services compile. Keep shared design tokens in `lib/app/theme.dart`.

### Editor

Create an `EditorControllerRegistry`/per-file editor state so each file has one stable `TextEditingController`, selection, and scroll controller. On file switch, swap existing controllers instead of constructing `TextEditingController(text: ...)` in `build`. Attach listeners for immediate dirty state and save. Implement search, select-all, undo/redo, horizontal scroll, monospace editing, line numbers/current-line affordance, and safe lightweight syntax highlighting (or use an existing mature package only if it is available and justified). Do not overlay a broken read-only highlighter.

### Preview

Create `ProjectBundleResolver` and `PreviewService`:

- detect the configured/first HTML entry file;
- resolve relative `link`, `script`, and image references where practical;
- return a useful `Entry file not found` state rather than synthesizing HTML;
- inject CSS/JS safely without fragile assumptions where possible;
- expose `PreviewStatus` (`idle`, `loading`, `ready`, `error`), reload and load error callbacks;
- keep WebView controller stable; reload only on explicit Run or configured auto-run-after-save debounce.

### Console

Create typed `ConsoleEntry`/`ConsoleLevel` and `ConsoleService` with bounded history. Capture log/info/warn/error, `window.onerror`, and unhandled errors supported by the bridge. Do not invent source locations. Separate system entries visually and provide clear.

### Execution

Refactor into `execution_provider.dart`, `process_event.dart`, and `unavailable_provider.dart`. Preserve the unavailable provider but change it to stream/event semantics with cancellation and command history. The UI must state: terminal provider unavailable; editing and web preview work, system commands do not.

### Tools

Replace the catalog-only `_toolsPanel` with real offline tool screens/services for JSON formatting, Base64 encode/decode, UUID generation, URL encode/decode, JWT viewing without signature verification, and timestamp conversion where feasible. Mark unimplemented tools disabled as `Coming soon`; remove `Available locally` from dead rows.

### App configuration/build foundation

Create `build/` abstractions (`BuildEngine`, request/result/event/status, preflight validator) and an honest unavailable Web APK engine unless a real build backend exists. Implement configuration editing and package ID validation. Preflight must block invalid configs and unavailable engines before any build attempt. Build history can record only actual engine results; no timer-based fake progress or fake APK output.

### Onboarding/error handling/accessibility

Add one-time onboarding persisted in settings; expose available vs coming-soon capabilities. Add `FlutterError.onError` and `PlatformDispatcher.instance.onError` bootstrap handling with user-friendly error surface/logging. Add semantic labels/tooltips to icon actions and ensure statuses are not color-only.

## Replace/remove

- Replace `_project` single in-memory workspace with repository-backed project collection and active project.
- Replace `_newProject` destructive reset with template creation flow (Blank, HTML Starter, Multi-page Web; Android/Flutter marked coming soon).
- Replace `_newFile` fixed `untitled.txt` with validated create-file flow and persistence.
- Replace editor `TextEditingController(text: _activeFile.content)` in `build` with stable per-file controllers.
- Replace `_save` flag-only behavior with repository save and dirty tracking.
- Replace `_run` permanent boolean with preview lifecycle and explicit `Run` event.
- Replace `PreviewFrame` fallback `No index.html` page with an error state and Choose Entry File action.
- Replace hard-coded preview asset names with bundle resolution.
- Replace string console lines with typed entries and bounded storage.
- Replace fake “Available locally” tools/settings toggles with working actions or disabled honest states.
- Replace navigation’s shared integer with a safe destination enum/index mapping.
- Replace settings no-op switches/dropdowns with persisted implemented settings; label unavailable features.
- Keep the Android release debug-signing fallback only as a clearly labeled development configuration; do not claim it is production signing.

## Implementation order

1. Extract models, serialization, repository/storage, settings, and templates; wire startup loading and persistence.
2. Extract navigation/theme and refactor shell to use repository-backed projects without changing the visual direction.
3. Fix editor controller lifecycle, dirty/save state, file operations, and project switching.
4. Add bundle resolver, preview lifecycle/error states, console bridge/types, and explicit Run behavior.
5. Implement tools that are genuinely offline; make remaining catalog entries unavailable/coming soon.
6. Add app configuration, validation, build abstractions, preflight, and honest unavailable build surface.
7. Add onboarding, error handling, accessibility labels, responsive/landscape improvements, and settings persistence.
8. Add unit/widget tests for critical model/repository/template/validation/navigation/editor/preflight behavior.
9. Inspect/correct Android label, internet permission for WebView, orientation/theme/splash; keep debug vs release distinction.
10. Run `flutter pub get`, `flutter analyze`, `flutter test`, and `flutter build apk --debug` if the toolchain is available; otherwise report exact blockers.

## Verification matrix

- Unit: project serialization, templates, create/rename/delete/duplicate, file operations, package ID and app config validation, bundle resolution, console bounds, settings persistence.
- Widget: shell/navigation, project creation, editor dirty/save state, file switching, preview error state, build preflight.
- Manual when Flutter is available: restart persistence, multiple project switching, relative preview assets, console log/warn/error, invalid package ID blocking, settings persistence, landscape/large text, no dead visible actions.
- Build: debug APK only if Flutter SDK + Android SDK/Gradle are available. A successful debug APK must be reported as such; no APK result is claimed otherwise.

## Expected V2 capability boundary

The V2 foundation can deliver local project editing, persistence, HTML/CSS/JS preview, console diagnostics, real offline tools, app configuration, and build preflight. Without a real local or remote build engine, Android/Flutter compilation and APK output remain explicitly unavailable. The UI must expose that state rather than simulate success.
