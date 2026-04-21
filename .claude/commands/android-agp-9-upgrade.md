Migrate the current Android project to Android Gradle Plugin (AGP) 9. Kotlin Multiplatform (KMP) projects are out of scope — stop and tell the user if this is a KMP project.

## Pre-flight check

Use Glob to find `build.gradle`, `build.gradle.kts`, and `libs.versions.toml`. Read them to determine the current AGP version.

If the current AGP version is **below 9**, tell the user:
> "Your project is on AGP X.Y.Z. Please run the AGP Upgrade Assistant in Android Studio first (Tools → AGP Upgrade Assistant) to reach the latest stable AGP, then invoke this skill again."
> 
> Optionally, if the user explicitly asks to proceed anyway, continue.

## Migration steps

Execute the following steps in order. After each step, verify the change compiles before continuing.

### Step 1 — Update KSP and Hilt versions

Search `libs.versions.toml` (or `build.gradle` files) for `ksp` and `hilt` version declarations.
- KSP must be **2.3.6 or higher**
- Hilt must be **2.59.2 or higher**

Update any versions that are below the required minimum.

### Step 2 — Migrate to Built-in Kotlin

AGP 9 uses the built-in Kotlin toolchain. Apply the migration described in the official guide:
- Remove explicit `kotlin-android` plugin declarations where AGP now manages them
- Remove `android.builtInKotlin=true` from `gradle.properties` if present (it is now the default and the flag is deprecated)
- Reference: AGP 9 release notes section on built-in Kotlin

### Step 3 — Migrate to the new AGP DSL

AGP 9 introduces DSL changes. Scan `build.gradle(.kts)` files for deprecated DSL usages and update them per the AGP 9.0 release notes and gradle-recipes examples.

Common changes include:
- `compileSdkVersion` → `compileSdk`
- `buildToolsVersion` removal (no longer required)
- Namespace moved out of `defaultConfig` into the `android` block top level

### Step 4 — Migrate kapt → KSP (or legacy-kapt)

Search all `build.gradle(.kts)` files for `kapt` usages.

For each annotation processor:
- If the processor has a KSP variant available, migrate to KSP (`ksp(...)` instead of `kapt(...)`)
- If no KSP variant exists, switch to `id("com.google.devtools.ksp")` with `legacy-kapt` as documented in the AGP 9 migration guide

Remove the `kotlin-kapt` plugin from any module that no longer uses it.

### Step 5 — Handle custom BuildConfig fields

Search for `buildConfigField(...)` calls. AGP 9 requires `buildFeatures { buildConfig = true }` to be explicitly set in any module that uses BuildConfig.

Add the `buildFeatures` block where missing.

### Step 6 — Clean gradle.properties

Remove the following deprecated flags from `gradle.properties` if present:
- `android.builtInKotlin`
- `android.newDsl`
- `android.uniquePackageNames`
- `android.enableAppCompileTimeRClass`

Also remove any temporary migration flags added by the AGP Upgrade Assistant.

## Verification

After all steps:
1. Trigger a Gradle sync (remind the user to do this in Android Studio)
2. Confirm the following commands succeed (tell the user to run them):
   ```
   ./gradlew help
   ./gradlew build --dry-run
   ```

## Paparazzi warning

If the project uses Paparazzi, check its version. Paparazzi **v2.0.0-alpha04 and earlier** have known compatibility issues with AGP 9. Warn the user and point them to the Paparazzi release notes for a compatible version.