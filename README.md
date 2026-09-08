# Vaani / VoiceGuard â€” Frontend Repo (UI-Only Workspace)

This repo is a **frontend-only mirror** of the `voice_guard` Flutter app from the
[`Smart_India_Hackathon`](https://github.com/VexBane777/Smart_India_Hackathon.git) repo
(`vaani` branch). Its purpose: let UI developers build and restyle the app's interface
**without any knowledge of the backend, native Android code, or backendâ†”frontend wiring**.

All backend wiring (services, providers, models, native Android glue) is present and
**byte-identical** to the main repo, so the app builds and runs exactly as-is â€” the UI
devs get a fully working app as their canvas, and any interface work can be pulled
straight back into the main repo's `vaani` branch with **zero manual stitching**.

---

## âœ… You MAY edit (the UI surface)

| Path | What it is |
|---|---|
| `lib/screens/` | All app screens (home, call, logs, settings, onboarding, VoIP protection) |
| `lib/widgets/` | Reusable UI components (risk meter, waveform, permission card, status indicator) |
| `lib/app.dart` | App shell: theme, colors, navigation bar, routing |
| `assets/images/`, `assets/sounds/` | Static UI assets |
| `test/` | Widget/UI tests |
| `pubspec.yaml` | **Only** to add *pure-UI* packages (fonts, icons, animations). Do not touch existing dependencies. |

## ðŸš« You must NOT edit (backend wiring â€” off limits)

| Path | Why |
|---|---|
| `lib/services/` | API, WebSocket/signaling, WebRTC, TFLite, audio pipeline, native platform channels |
| `lib/providers/` | State providers that services push data into â€” the UI *consumes* these, never rewrites them |
| `lib/models/` | Data contracts shared with the backend (risk score, call state, call log) |
| `lib/utils/` | Constants, permission helpers, audio processing |
| `lib/main.dart` | Bootstrap: wires services â†’ providers â†’ app. Touching this breaks the wiring |
| `android/` | Native Kotlin (call detection, audio capture, privileged permissions) |
| `pubspec.yaml` native/IO dependencies (`tflite_flutter`, `flutter_webrtc`, `web_socket_channel`, `permission_handler`, â€¦) | These ARE the backend wiring |

Enforcement: a `CODEOWNERS` file marks the off-limits paths â€” enable **branch protection
+ required code-owner review** on GitHub to mechanically block changes to them.

---

## How the wiring works (the only thing you need to know)

The app uses the `provider` package. `main.dart` creates services and exposes them as
providers. Your screens/widgets **read** state from providers and **call** their public
methods â€” you never create or modify them:

```dart
// Read reactive UI state (rebuilds your widget automatically):
final risk = context.watch<RiskScoreProvider>();   // .riskLevel, .confidence, .isScoring
final call = context.watch<CallStateProvider>();   // .status, .phoneNumber

// Trigger backend actions (read-only usage of services):
context.read<AudioService>().startScoring();
context.read<CallService>().setCallStateCallback(...);

// Persisted settings:
final settings = context.read<SettingsProvider>();
```

**Contract:** as long as providers keep their existing getters/methods and screens keep
consuming them the same way, the UI can be redesigned freely and will merge back cleanly.

## Run it

```bash
flutter pub get
flutter run            # on an Android device/emulator (native call/audio features need one)
flutter analyze        # must pass before opening a PR
```

## Syncing back to the main repo

Copy/replace `lib/`, `assets/`, `test/`, and any *added UI files* from this repo into
`voice_guard/` on the `vaani` branch of the main repo. Because `lib/services`,
`lib/providers`, `lib/models`, `lib/utils`, `lib/main.dart`, and `android/` are unchanged
by policy, git merges will be conflict-free (or trivially resolvable).

## For maintainers (repo owner)

To refresh this mirror after new `vaani` commits:

```bash
git -C C:\path\to\SIH archive -o export.tar vaani voice_guard
tar -xf export.tar -C C:\path\to\SIH_frontend --strip-components=1
# then delete backend/, model_training/, magisk-privileged-module/, docs/ if present
```
