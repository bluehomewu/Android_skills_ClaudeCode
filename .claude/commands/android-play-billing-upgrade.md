Upgrade the Google Play Billing Library (PBL) to the latest stable version. This skill handles both direct migrations (within 2 major versions) and stepped migrations (3+ major versions behind).

Target version (leave blank for latest): $ARGUMENTS

## Phase 0 — Announce intent

Before touching any files, tell the user:
> "I will upgrade your Play Billing Library to [target version / the latest stable version]. I'll start by discovering your current version, then plan and execute the migration."

## Phase 1 — Discovery

### 1a — Find current version

Use Glob to find all `build.gradle`, `build.gradle.kts`, and `libs.versions.toml` files. Read them and locate the Play Billing Library dependency:
- `com.android.billingclient:billing` or `com.android.billingclient:billing-ktx`

Record the **declared version**.

### 1b — Verify effective version

If the build uses BOMs or version catalogs that might override the declared version, trace through to find the effective version being compiled.

### 1c — Identify deprecated APIs (if build fails)

If the project currently fails to compile due to PBL API changes, Grep the source for deprecated symbols from known PBL major versions:
- PBL 5→6: `BillingFlowParams.Builder.setSkuDetails()` removed → `setProductDetailsParamsList()`
- PBL 6→7: `SkuDetails`, `Purchase.getPurchaseState()` changes
- PBL 7→8: `queryProductDetailsAsync` result type changes

Use the presence of deprecated symbols to infer the minimum "effective version" the code was written against.

### 1d — Determine migration strategy

Calculate the version delta:
- **Direct migration** (≤ 2 major versions apart): upgrade in one step
- **Stepped migration** (> 2 major versions apart): upgrade two major versions at a time, verifying compilation at each step

Report the migration path to the user before proceeding.

## Phase 2 — Build the migration plan

For each major version jump in the migration path, synthesize the required changes from the official release notes and migration guide:

PBL major version migration highlights (apply only the relevant ones):

**→ v5**: Replace `SkuDetails` with `ProductDetails`. Replace `querySkuDetailsAsync` with `queryProductDetailsAsync`.

**→ v6**: Update `BillingFlowParams` construction to use `ProductDetailsParams`. Handle new `PENDING` purchase state explicitly.

**→ v7**: Update `acknowledgePurchase` / `consumeAsync` signatures. Handle `BillingResult` changes.

**→ v8+**: Check official release notes for current breaking changes.

List the exact files and methods that need changing before making any edits. Get user confirmation.

## Phase 3 — Execute

### For direct migration:

1. Update the PBL version in `libs.versions.toml` or `build.gradle(.kts)`.
2. Apply all API changes identified in Phase 2 using Edit.
3. Resolve any remaining compilation errors by reading them and fixing the root cause (do not suppress).

### For stepped migration:

For each intermediate version:
1. Update the version.
2. Apply the API changes for that version bump only.
3. Tell the user: "Please run `./gradlew compileDebugKotlin` to verify this intermediate version compiles before I continue."
4. Wait for confirmation, then proceed to the next version step.

## Phase 4 — Validation

After reaching the target version:

1. **Version checklist**: Confirm the following for the target version:
   - Dependency version updated ✓
   - All deprecated APIs replaced ✓
   - `BillingClient` connection lifecycle handling matches new requirements ✓
   - Purchase acknowledgement logic updated if required ✓

2. **Test reminder**: Tell the user:
   > "Run your unit and integration tests. If you have billing integration tests, run them against a test account in the Play Console sandbox. Run `./gradlew clean assembleDebug` for a full clean build."

3. **Explain changes**: After all edits, summarise every significant change made:
   - What was changed
   - Why (reference the deprecation or breaking change from the release notes)
   - Any new features now available in this PBL version that the app could benefit from
