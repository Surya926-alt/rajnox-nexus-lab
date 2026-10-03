# NEXUS LAB

## V2.1 stabilization

This build is based on the existing V2 project and fixes the stabilization pass without replacing its working architecture:

- Relative HTML/CSS/JavaScript and inline image assets resolve through `ProjectBundleResolver`.
- Preview reloads use a revision token and expose `IDLE`, `LOADING`, `READY`, and `ERROR` states.
- Console bridge serializes objects and captures `log`, `info`, `warn`, `error`, `window.onerror`, and unhandled promise rejections.
- Terminal input is locked when no execution provider is connected.
- Base64 and URL tools support both encode and decode.
- Autosave is persisted and debounced after editing stops; optional auto-run is also debounced.
- `AppConfig` persists icon, splash, splash animation, orientation, fullscreen, system bars, and permissions.
- Project persistence is cached, guarded against malformed storage, and supports create/open/rename/delete/duplicate/switch flows.
- APK preflight exposes individual checks; the build engine remains an honest unavailable abstraction and never fabricates an APK.

Flutter verification was not run in the source sandbox because the Flutter SDK is unavailable there.

**BUILD. RUN. EXPLORE.**

A dark, responsive Flutter developer workspace for Android-first development. V2 prioritizes a real HTML/CSS/JS live lab, editable project files, restart-safe project persistence, and explicit provider boundaries for capabilities that require native Android integration.

## V2 foundation included

- Premium graphite/violet developer workspace UI
- Responsive phone/tablet/desktop layouts
- Repository-backed project/file models with structured JSON persistence
- Blank, HTML starter, and multi-page project templates
- Mobile-friendly editor with save state and file switching
- Embedded `webview_flutter` live preview
- JavaScript `console.log` bridge into the in-app console
- Project creation, switching, file creation/rename/delete, and restart-safe local workspace state
- Terminal shell UI with command history output and an explicit unavailable execution state
- `ExecutionProvider`, `ProcessEvent`, and `AIProvider` abstractions
- Command palette / floating command button
- Real offline JSON, Base64, URL, and UUID utilities
- System Monitor page that shows `Unavailable on this device` instead of invented telemetry
- Settings page with app configuration, editor preferences, and execution status
- Smoke test for the main app shell

## Honest limitations

- The terminal does not execute arbitrary commands yet. It routes through a deliberately unavailable `ExecutionProvider` rather than faking output.
- APK compilation is not connected. The build abstraction and preflight validation exist, but no APK is claimed or produced without a real engine.
- Native Android telemetry and sandboxed execution require platform-specific providers and permissions.
- The AI panel is represented by an `AIProvider` interface and does not contain API keys or a hardcoded provider.

## Run

```bash
export PATH=/home/ubuntu/flutter/bin:$PATH
cd /home/ubuntu/projects/nexus_lab
flutter pub get
flutter test
flutter analyze --no-fatal-infos
flutter run
```

For an Android build, run from a machine with the Android SDK configured:

```bash
flutter build apk --debug
```

## V2 architecture

The former monolithic shell now delegates core responsibilities to:

- `ExecutionProvider`
- `ProjectRepository` / `ProjectStorage`
- `ProjectBundleResolver`
- `BuildEngine` / `BuildPreflight`
- `ProcessEvent` / `ExecutionProvider`
- `AIProvider`
- System monitor surface with an explicit unavailable state until a native provider exists

See [`V2_IMPLEMENTATION_PLAN.md`](V2_IMPLEMENTATION_PLAN.md) for the file/class mapping and preserved/refactored/replaced V1 decisions.
