# All-in-One Student Toolkit

Android app scaffold — Kotlin + Jetpack Compose + Room (offline, no paid APIs).

## What's already working

Open this folder in **Android Studio** (Hedgehog or newer), let Gradle sync, and hit Run.
Four tools are fully functional out of the box:

- **GPA / CGPA Calculator** — add courses with credit hours + letter grade, saved to Room, live CGPA.
- **Attendance Tracker** — mark present/absent per subject, running percentage per subject.
- **Calculator** — standard arithmetic (uses exp4j for safe expression evaluation).
- **Unit Converter** — length, weight, temperature.
- **Dark mode** — toggle on the home screen (persisted with DataStore), or it follows the system setting by default.

Everything else (Study Planner, Timetable, Notes, Assignment Manager, Pomodoro Timer,
PDF Scanner, QR Scanner) shows a "Coming soon" placeholder and is already wired into
navigation — see `navigation/NavGraph.kt`.

## Project layout

```
app/src/main/java/com/studenttoolkit/app/
├── MainActivity.kt              # entry point: sets up theme, DB, nav
├── navigation/
│   ├── Destinations.kt          # one entry per tool (add new tools here first)
│   └── NavGraph.kt              # routes screens together
├── data/
│   ├── AppDatabase.kt           # Room database (offline, on-device SQLite)
│   ├── SettingsRepository.kt    # DataStore: dark mode preference
│   ├── entities/                # CourseGrade, AttendanceRecord, Note, Assignment, TimetableEntry
│   └── dao/                     # matching DAOs
└── ui/
    ├── theme/                   # Color.kt, Theme.kt (light/dark schemes)
    └── screens/                 # one file per tool screen
```

## Adding the next tool (e.g. Notes)

The `Note` entity and `NoteDao` already exist in `data/`. To wire up the screen:

1. Create `ui/screens/NotesScreen.kt` — copy the structure of `GpaCalculatorScreen.kt`
   (it's the simplest Room-backed example: `collectAsStateWithLifecycle` to read,
   `rememberCoroutineScope().launch { dao.insert(...) }` to write).
2. In `NavGraph.kt`, remove `Destination.NOTES` from the placeholder `listOf(...)` block
   and add its own `composable(Destination.NOTES.route) { NotesScreen(...) }` entry,
   following the pattern used for GPA/Attendance above it.
3. In `Destinations.kt`, flip `implemented = false` to `true` for `NOTES`.

Repeat for Study Planner, Timetable, and Assignment Manager — they all follow the same
entity → DAO → screen → nav-graph pattern.

## Pomodoro Timer

Not Room-backed — just in-memory state + a `CountDownTimer`. Consider running it in a
foreground `Service` so the timer survives the screen turning off. No new dependencies needed.

## PDF Scanner / QR Scanner (on-device, free, no API key)

Uncomment these lines in `app/build.gradle.kts` when you build these screens:

```kotlin
implementation("com.google.mlkit:barcode-scanning:17.3.0")                          // QR Scanner
implementation("com.google.android.gms:play-services-mlkit-document-scanner:16.0.0-beta1") // PDF Scanner
implementation("androidx.camera:camera-core:1.3.4")
implementation("androidx.camera:camera-camera2:1.3.4")
implementation("androidx.camera:camera-lifecycle:1.3.4")
implementation("androidx.camera:camera-view:1.3.4")
```

Both ML Kit APIs run entirely on-device — no internet connection or API key required,
so this keeps the "works offline, no paid API" requirement intact.

## AdMob (monetization)

1. Create an AdMob account and app entry, get your App ID and ad unit IDs.
2. Uncomment the AdMob dependency in `app/build.gradle.kts`:
   `implementation("com.google.android.gms:play-services-ads:23.2.0")`
3. Uncomment the `<meta-data>` block in `AndroidManifest.xml` and paste your real App ID.
4. Add a banner ad to the bottom of `HomeScreen.kt` (or wherever you like) using
   `AndroidView` to host a Compose-wrapped `AdView`. Save interstitials for natural
   breakpoints (e.g. after finishing a Pomodoro session) rather than between every tap —
   it's better for both UX and AdMob policy compliance.
5. **Use test ad unit IDs during development** (Google publishes these) so you don't
   risk your AdMob account for accidental clicks on real ads.

## Launcher icon

`drawable/ic_launcher_foreground.xml` is a placeholder shape. Before publishing, replace
it with a real icon — in Android Studio: right-click `res` → New → Image Asset.

## App icon / branding, screenshots, Play Store listing

Not included here — those are store-listing assets, not code. Happy to help with app
naming, description copy, or a privacy policy (required by AdMob) once the app is closer
to done.
