Analyze the Android project's R8/ProGuard keep rules and build configuration to identify redundancies, overly broad rules, and optimizations that reduce APK size.

## Constraints

- **Do not modify** any keep rule files directly during analysis — produce recommendations only.
- Do not mention internal tooling names or hierarchy level labels in the report.
- Do not generate empty report sections.
- Organize all findings by impact level (highest impact first).

## Step 1 — Create analysis report

Create a file `r8-analysis-report.md` at the project root. All findings will be written here incrementally as the analysis progresses.

## Step 2 — Review build configuration

Use Glob to find:
- All `build.gradle` / `build.gradle.kts` files
- `proguard-rules.pro` / `consumer-rules.pro` files
- `libs.versions.toml`

Read each file. Record in the report:
- Current AGP version
- Whether `isMinifyEnabled = true` is set for the release build type
- Whether `isShrinkResources = true` is set
- Whether `proguard-android-optimize.txt` is used as the base rules file
- All custom keep rule files referenced

## Step 3 — Assess AGP version

If the project is on AGP below 9.0, add a recommendation:
> "Upgrading to AGP 9.0+ enables built-in R8 full mode and additional dead-code elimination. Consider using /android-agp-9-upgrade to migrate."

## Step 4 — Evaluate keep rules — Phase A (library consumer rules)

Read all `proguard-rules.pro` files. For each rule, check whether the rule is already provided by the library's bundled consumer rules (i.e., it is redundant when the library is a dependency).

Common redundant rules that libraries already ship:
- OkHttp / okio keep rules (bundled since OkHttp 4.x)
- Retrofit interface keep rules (bundled since Retrofit 2.6)
- Gson `@SerializedName` rules (bundled)
- kotlinx.serialization rules (bundled via the Gradle plugin)
- Ktor client rules (bundled)
- Compose runtime rules (bundled via AGP Compose integration)

For each rule confirmed redundant, add to the report:
```
[REMOVE] <rule text>
Reason: Already provided by <library> consumer rules.
Estimated impact: Low / Medium / High
```

## Step 5 — Evaluate keep rules — Phase B (impact assessment)

For remaining rules, assess each by the following criteria:

1. **Overly broad package wildcards** (`-keep class com.example.**`) — these prevent tree-shaking entire packages. Flag for narrowing.
2. **`-keep` vs `-keepclassmembers`** — if only specific members need to survive, `-keep` retains the class *and* all members unnecessarily.
3. **Redundant subsumption** — if rule A covers everything rule B covers, rule B is redundant.
4. **`-dontwarn` for classes that no longer exist** — can be removed.

For each finding, record in the report:
```
[REMOVE / REFINE] <rule text>
Reason: <explanation>
Suggested replacement (if REFINE): <narrower rule>
Estimated impact: Low / Medium / High
```

## Step 6 — Code impact analysis

For rules flagged as candidates for removal or refinement, Grep the source tree to check if the kept class/member is accessed via reflection (e.g., `Class.forName`, `getDeclaredMethod`, `getDeclaredField`).

If reflection is found:
- Keep the rule but narrow it to the specific class or member accessed
- Document the reflection call site in the report

If no reflection is found and the class is only referenced through normal Kotlin/Java code:
- R8 can track it statically; the keep rule may be safely removed
- Mark as `[REMOVE - safe]`

## Step 7 — Priority ordering

Reorder all findings in `r8-analysis-report.md` from highest estimated APK size impact to lowest.

## Step 8 — Summary

Add a summary section at the top of the report:
- Total rules analysed
- Rules recommended for removal
- Rules recommended for refinement
- Estimated combined APK size reduction (rough estimate based on scope of rules)

## Step 9 — Testing guidance

Add a final section to the report:
> **Validation steps after applying recommendations:**
> 1. Apply changes incrementally — one rule at a time, or by library group.
> 2. Run a release build: `./gradlew assembleRelease`
> 3. Test all app flows, especially those involving network, serialization, or reflection-heavy libraries.
> 4. Use `./gradlew :app:generateReleaseR8Usage` to confirm removed classes are genuinely absent.

Tell the user the report is complete and show them the path to `r8-analysis-report.md`.
