Migrate one Android XML layout to Jetpack Compose using a structured 10-step incremental approach. Focus on a single screen or component at a time — do not attempt to migrate the entire app in one pass.

The target XML file or component to migrate: $ARGUMENTS

## Constraints

- Migrate **one XML candidate** per invocation. Do not rewrite the entire theme or unrelated screens.
- Preserve all existing project naming conventions, code style, and patterns.
- Do not remove any file until you have confirmed nothing else references it.
- Only migrate UI layer code — do not touch business logic, ViewModels, or data handling.

## Step 1 — Identify the XML candidate

If `$ARGUMENTS` names a specific file, use it. Otherwise, use Glob to find `res/layout/*.xml` files and select the best candidate by these criteria (in order):
1. No inheritance from other layouts (no `<include>` that would create a cascade)
2. Self-contained: all data comes from one ViewModel or is static
3. No existing Compose dependency (avoid double-migration)

Read the chosen XML file fully and report its name and complexity to the user before proceeding.

## Step 2 — Analyse the layout

Read the XML file and produce an analysis:
- List every View/ViewGroup type used
- Note any custom views, data binding expressions, or view IDs referenced in Kotlin/Java code
- Grep the source tree for all usages of those view IDs to understand the call sites

## Step 3 — Plan the migration

Based on the analysis, describe:
- The target composable function name and file location
- How each XML view maps to a Compose equivalent
- Any interop needed (`AndroidView`, `ComposeView`) for custom views without Compose equivalents
- Which call sites need updating after the composable is created

Get user confirmation before proceeding.

## Step 4 — Capture a visual baseline

Tell the user:
> "Before writing any code, please take a screenshot of the current UI for this screen and save it. This will be used for visual validation in Step 9."

## Step 5 — Verify Compose dependencies

Read `build.gradle(.kts)` and `libs.versions.toml`. Confirm the following are present:
- `androidx.compose.bom` platform dependency
- `androidx.activity:activity-compose`
- `androidx.compose.ui:ui`, `androidx.compose.material3:material3`
- `kotlinOptions { jvmTarget = "11" }` (or higher)
- `buildFeatures { compose = true }`

Add any missing dependencies and sync before continuing.

## Step 6 — Set up minimal Compose theming

Do **not** migrate the full app theme. Only add the minimum required for this component:
- If a `MaterialTheme` wrapper already exists in the app, reuse it
- If not, create a minimal theme wrapper only for this composable's preview

## Step 7 — Write the composable

Create the new `.kt` file with the composable function:
- Match the visual layout of the original XML precisely
- Add a `@Preview` annotated function
- Use `Modifier` parameters to allow the caller to control padding/size
- Implement all click listeners and state that were handled in the original View code

## Step 8 — Update call sites

Grep for all locations that inflate, reference, or set the original XML layout. Update each call site to use the new composable via `setContent { }` or `ComposeView` as appropriate.

## Step 9 — Visual validation

Tell the user:
> "Please run the app, navigate to this screen, and compare it against your baseline screenshot. Report any visual differences."

If discrepancies are found, fix the composable and repeat. Write a UI test for the composable using `ComposeTestRule`.

## Step 10 — Remove legacy files

Only after the user confirms visual parity and all call sites are updated:
- Delete the original XML layout file
- Delete any associated XML test files that only covered the removed layout
- Remove unused imports in files that referenced the old layout

Confirm with the user before deleting each file.
