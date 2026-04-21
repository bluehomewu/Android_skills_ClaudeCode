Help implement or migrate Android navigation using Jetpack Navigation 3. This covers: new Navigation 3 setup, migration from Navigation 2, and implementing specific Navigation 3 patterns.

Task or pattern to implement: $ARGUMENTS

## Determine the mode

Read the project's `build.gradle(.kts)` and `libs.versions.toml` to identify which navigation library is currently used:

- **No navigation library**: Set up Navigation 3 from scratch → follow "New Setup"
- **Navigation 2.x** (`androidx.navigation:navigation-compose`): → follow "Migration from Nav 2"
- **Navigation 3 already present**: → follow "Pattern Implementation" for `$ARGUMENTS`

---

## New Setup

### 1 — Add dependencies

Add to `libs.versions.toml`:
```toml
[versions]
navigation3 = "1.0.0-alpha01"   # use latest stable

[libraries]
androidx-navigation3-compose = { group = "androidx.navigation3", name = "navigation3-compose", version.ref = "navigation3" }
```

Add to `build.gradle.kts`:
```kotlin
implementation(libs.androidx.navigation3.compose)
```

### 2 — Define type-safe routes

Create a sealed class or `@Serializable` data classes/objects for all destinations:
```kotlin
@Serializable data object Home
@Serializable data class Detail(val id: String)
```

### 3 — Create the NavDisplay

Set up `NavDisplay` in your `Activity` or root composable with a `backStack` and entry providers:
```kotlin
val backStack = rememberNavBackStack(Home)
NavDisplay(backStack = backStack) {
    entry<Home> { HomeScreen(onNavigate = { backStack.add(Detail(it)) }) }
    entry<Detail> { DetailScreen(id = it.id) }
}
```

### 4 — Wire navigation actions

Pass `backStack.add(...)` / `backStack.removeLastOrNull()` down to screens via lambda parameters (do not pass the backStack directly to screens).

---

## Migration from Navigation 2

### 1 — Update dependencies

Remove `androidx.navigation:navigation-compose` and add `navigation3-compose` as above.

### 2 — Replace NavHost

Remove `NavHost { composable("route") { ... } }` blocks. Replace with `NavDisplay` and type-safe route objects.

### 3 — Replace NavController

Replace all `navController.navigate("route")` calls with `backStack.add(RouteObject)`. Replace `navController.popBackStack()` with `backStack.removeLastOrNull()`.

### 4 — Replace route strings

Convert all string-based routes to `@Serializable` data classes/objects. Update `NavArgument` usages to constructor parameters on the route class.

### 5 — Update result passing

Replace `SavedStateHandle` result passing with shared ViewModel or state-based approach documented in the Navigation 3 results recipes.

### 6 — Verify deep links

Update any deep link handling to use the Navigation 3 deep link API.

---

## Pattern Implementation

Implement the specific pattern from `$ARGUMENTS`. Supported patterns:

| Pattern | Description |
|---|---|
| `bottom-nav` | Bottom navigation with separate back stacks per tab |
| `deep-links` | Basic and advanced deep link wiring |
| `multiple-backstacks` | Independent back stacks with state persistence |
| `conditional` | Auth/onboarding conditional navigation flows |
| `list-detail` | Material 3 Adaptive list-detail layout |
| `dialog` | Dialog and bottom sheet destinations |
| `pass-arguments` | Type-safe argument passing between destinations |
| `results` | Passing results back from a destination |
| `animations` | Custom enter/exit transition animations |

Read the existing navigation code in the project, then implement the requested pattern following Navigation 3 conventions. Show the implementation with before/after diffs where code is being changed.

---

## Verification

After any change:
1. Confirm Gradle sync succeeds
2. Check for any remaining `NavController` or `NavHost` references (for migration) using Grep
3. Remind the user to test back-stack behaviour manually on device/emulator
