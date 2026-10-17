# Releasing USSI

## Maven coordinates

Published artifacts use **`com.uber.ussi:ussi`**.

## CI and local builds

- **Pull requests and main:** Bazel tests (`.github/workflows/tests.yml`).
- **Releases:** `.github/workflows/release.yml` (Gradle `publishAndReleaseToMavenCentral`, GitHub Release, changelog).

Both build systems share the same `src/main/java` and `src/test/java` trees. Bazel remains supported for day-to-day development; Gradle is the supported release path for Uber open-source JVM libraries.

Gradle requires JDK 21 or later. On Uber-managed machines that inject an internal Artifactory `init.gradle`, use a clean `GRADLE_USER_HOME` (for example `GRADLE_USER_HOME=$PWD/.gradle-home ./gradlew test`) so dependencies resolve from Maven Central only.

## Version files

Keep these in sync:

- `gradle.properties` — `VERSION_NAME` (use `-SNAPSHOT` on `main` between releases)
- `BUILD.bazel` — `maven_coordinates` on `//:src_main`
- `CHANGELOG.md` — `## [X.Y.Z]` section for each release

## Cut a release (recommended: workflow dispatch)

1. On `main`, set `VERSION_NAME` to the next release with a `-SNAPSHOT` suffix (for example `0.2.0-SNAPSHOT`) and align `BUILD.bazel`.
2. Add a `## [0.2.0]` section to `CHANGELOG.md`.
3. Merge to `main`.
4. Run **Release to Maven Central** → **Run workflow** on `main` (optional `release_version` override).
   - CI strips `-SNAPSHOT`, commits the release version to `gradle.properties` and `BUILD.bazel`, publishes to Maven Central, and creates the GitHub Release + tag.
5. Confirm the artifact on [Maven Central](https://central.sonatype.com/artifact/com.uber.ussi/ussi).
6. Open a follow-up PR bumping `VERSION_NAME` and `BUILD.bazel` to the next `-SNAPSHOT` (branch protection may block the workflow from doing this automatically).

## Cut a release (tag push)

Alternatively, after `VERSION_NAME` and `BUILD.bazel` already hold the release version (no `-SNAPSHOT`):

1. Push annotated tag `vX.Y.Z`.
2. The same workflow tests, publishes, and creates the GitHub Release.

If tag `vX.Y.Z` already exists on an older commit, move the tag after fixing the release workflow, or cut the next patch version.
