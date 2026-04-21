Add adaptive edge-to-edge support to Android Jetpack Compose screens so the UI renders behind system bars (status bar, navigation bar) and handles the IME (soft keyboard) correctly.

Screen or Activity to update (leave blank to audit the whole project): $ARGUMENTS

## Requirements check

Read `build.gradle(.kts)` and `libs.versions.toml`. Verify:
- The project uses **Jetpack Compose** — if not, stop and tell the user this skill only supports Compose projects.
- `targetSdk` is **35 or higher** — if not, recommend updating it and ask the user to confirm before proceeding.

## Phase 1 — Planning

### 1a — Find Activities lacking edge-to-edge

Grep the source tree for all `ComponentActivity` subclasses. Read each one and check:
- Does it call `enableEdgeToEdge()` before `setContent`? (flag if missing)
- Does its `AndroidManifest.xml` entry have `android:windowSoftInputMode="adjustResize"`? (flag if it uses a soft keyboard but is missing this attribute)

### 1b — Find UI components that need inset handling

Grep for:
- `LazyColumn`, `LazyRow`, `LazyVerticalGrid` — need `contentPadding` with insets
- `FloatingActionButton` — usually needs `navigationBarsPadding()`
- `TextField`, `OutlinedTextField` — need IME padding analysis
- `Scaffold` — check whether `contentWindowInsets` is overridden

Report the full audit to the user before making any changes.

## Phase 2 — Enable edge-to-edge

For each Activity identified in Phase 1a:

### 2a — Call enableEdgeToEdge()

In `onCreate`, add `enableEdgeToEdge()` **before** `setContent { }`:
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    enableEdgeToEdge()   // ← add this
    setContent { ... }
}
```

Import: `androidx.activity.enableEdgeToEdge`

### 2b — Update AndroidManifest.xml

For Activities that host screens with text input fields, add to the `<activity>` element:
```xml
android:windowSoftInputMode="adjustResize"
```

## Phase 3 — Apply insets to Compose UI

For each flagged component from Phase 1b, apply the correct inset strategy:

### Scaffold (preferred pattern)

Ensure the `Scaffold` passes its `PaddingValues` to the content and does not add extra padding on top:
```kotlin
Scaffold { innerPadding ->
    Content(modifier = Modifier.padding(innerPadding))
}
```

**Do not** apply both `Scaffold` padding and additional `safeDrawingPadding()` to the same content — this causes double padding.

### Lists (LazyColumn / LazyRow)

Pass insets to `contentPadding`, not to the parent container:
```kotlin
LazyColumn(
    contentPadding = WindowInsets.safeDrawing.asPaddingValues()
) { ... }
```

### Standalone components without Scaffold

Apply `Modifier.safeDrawingPadding()` to the outermost composable that needs to avoid system bars.

### IME (soft keyboard) handling

**Preferred approach** — use `fitInside` with IME ruler (less layout jank):
```kotlin
Modifier.fitInside(WindowInsetsRulers.Ime.current)
```

**Alternative** — use `imePadding()` modifier, placed **before** `verticalScroll`:
```kotlin
Modifier
    .imePadding()           // ← before scroll
    .verticalScroll(state)
```

When a `Scaffold`'s `contentWindowInsets` already includes IME insets, do **not** also add `imePadding()` inside the content — this doubles the padding.

### Full-screen dialogs

For dialogs that should render edge-to-edge:
```kotlin
Dialog(
    properties = DialogProperties(decorFitsSystemWindows = false)
) { ... }
```

### Adaptive Scaffolds (multi-pane layouts)

Apply insets to **individual screens**, not to the shared scaffold parent:
```kotlin
// ✗ Wrong — applies insets to both panes at once
AdaptiveLayout(modifier = Modifier.safeDrawingPadding()) { ... }

// ✓ Correct — each pane manages its own insets
ListPane(modifier = Modifier.safeDrawingPadding()) { ... }
DetailPane(modifier = Modifier.safeDrawingPadding()) { ... }
```

### System bar appearance (light/dark)

If using `ComponentActivity.enableEdgeToEdge()`, system bar appearance adapts automatically.

If using `WindowCompat.setDecorFitsSystemWindows(window, false)` manually, you must also set:
```kotlin
WindowCompat.getInsetsController(window, view).apply {
    isAppearanceLightStatusBars = !isDarkTheme
    isAppearanceLightNavigationBars = !isDarkTheme
}
```

## Phase 4 — Verification

After all changes:

1. Tell the user to run the app and scroll to the bottom of lists — content should be visible above the navigation bar.
2. Tap on any text field — the keyboard should push content up without overlapping.
3. Check the status bar area — app content should extend behind it without text being clipped.
4. Test on both light and dark themes to confirm status/nav bar icon colours are correct.

If any visual issue is found, re-read the affected composable and fix the inset strategy.
