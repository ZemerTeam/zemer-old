# Repository Guidelines

## Project Structure & Module Organization
Metrolist is a Gradle workspace. `app/` hosts the Compose client; production code lives in `app/src/main`, dev helpers in `app/src/debug`, generated Room schemas under `app/schemas`, and APK/AAB artifacts under `app/build/outputs/`. Networking helpers are split into `innertube/` (YouTube extractor facade) and `lrclib/` (lyrics providers), each with its own `src/main` plus JUnit tests. Shared dependency versions stay in `gradle/libs.versions.toml`, while static media and store copy live in `assets/` and `fastlane/`.

## Build, Test, and Development Commands
Use the Gradle wrapper from repo root:
- `./gradlew assembleRelease` - produces the shrinked release artifacts in `app/build/outputs/`.
- `./gradlew :app:installDebug` - deploys the persistent-debug signed build to an attached device or emulator.
- `./gradlew lintDebug` - Android Lint configured by `app/lint.xml`.
- `./gradlew :innertube:test :lrclib:test` - JVM tests for the service modules.
- `./gradlew connectedDebugAndroidTest` - Compose/UI instrumentation, requires a running emulator.

## Coding Style & Naming Conventions
Target Kotlin 2.2/Compose with JVM 21. Use four-space indentation, prefer trailing commas, and group files by feature (for example `feature/search/{data,ui}`). Compose entry points are named `FeatureScreen`, state containers `FeatureUiState`, and preview-only composables should carry `@Preview` metadata. ViewModels sit in `app/.../ui`, dependency-injection modules in `di/`. Keep resource IDs `snake_case`, classes UpperCamelCase, properties lowerCamelCase, and constants SCREAMING_SNAKE_CASE. Refresh generated Room/Hilt sources via `./gradlew :app:kspDebugKotlin` instead of committing outputs.

## Testing Guidelines
Add fast-running verifiers under `<module>/src/test/kotlin` with descriptive `FeatureNameTest` classes; innertube and lrclib already depend on JUnit4. Device-bound logic belongs in `app/src/androidTest/kotlin`. Store fixtures in `app/src/test/resources`. After schema or mapper changes, run `./gradlew :app:testDebugUnitTest` to validate Room migrations, then `connectedDebugAndroidTest` when UI or playback flows change.

## Commit & Pull Request Guidelines
History favors imperative, one-line summaries (for example `Remove PR analysis documentation files` or `app: Fix playback overlay`). Keep the subject at or below 60 characters, expand on motivation and linked issues in the body, and credit collaborators only when relevant. Each pull request should outline touched modules, list the verification commands you executed (`lintDebug`, `testDebugUnitTest`, emulator model), and attach screenshots or recordings for UI work. Call out release-version or asset changes and include reviewer-ready notes when bumping `versionCode`.

## Security & Configuration Tips
Release signing expects `app/keystore/release.keystore` plus `STORE_PASSWORD`, `KEY_ALIAS`, and `KEY_PASSWORD` environment variables; never edit or commit `persistent-debug.keystore`. Keep `local.properties`, API tokens, and emulator settings local. If secrets rotate, wipe artifacts with `./gradlew clean` before creating a new build to avoid leaking credentials.
