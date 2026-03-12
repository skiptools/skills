# CI/CD for Skip Projects

## GitHub Actions

### Full Workflow

```yaml
name: Skip CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  JAVA_VERSION: '17'

jobs:
  test:
    name: Build and Test
    runs-on: macos-14
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'

      - name: Install Skip
        run: brew install skiptools/skip/skip

      - name: Verify environment
        run: skip doctor

      - name: Build
        run: swift build

      - name: Test
        run: swift test

  export:
    name: Export Artifacts
    needs: test
    runs-on: macos-14
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'

      - name: Install Skip
        run: brew install skiptools/skip/skip

      - name: Export release
        run: skip export -c release -o ./artifacts

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: skip-artifacts
          path: ./artifacts/
          retention-days: 30
```

### Environment Variables

Set these as repository secrets for Android signing:

| Secret | Purpose |
|--------|---------|
| `KEYSTORE_BASE64` | Base64-encoded keystore file |
| `KEYSTORE_PASSWORD` | Keystore password |
| `KEY_ALIAS` | Key alias |
| `KEY_PASSWORD` | Key password |

Decode the keystore in CI:

```yaml
- name: Decode keystore
  run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > Android/app/keystore.jks
```

### Caching

Speed up builds by caching Gradle and Swift dependencies:

```yaml
- name: Cache Gradle
  uses: actions/cache@v4
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: gradle-${{ runner.os }}-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}

- name: Cache Swift packages
  uses: actions/cache@v4
  with:
    path: .build
    key: swift-${{ runner.os }}-${{ hashFiles('Package.resolved') }}
```

## Fastlane Integration

For advanced deployment workflows, Skip projects work with [Fastlane](https://fastlane.tools):

```ruby
# Fastfile
platform :ios do
  lane :release do
    build_app(workspace: "Project.xcworkspace", scheme: "MyApp")
    upload_to_app_store
  end
end

platform :android do
  lane :release do
    gradle(task: "bundleRelease", project_dir: "Android")
    upload_to_play_store(aab: "Android/app/build/outputs/bundle/release/app-release.aab")
  end
end
```

## Running Tests on Android in CI

For instrumented tests in CI, you need an Android emulator:

```yaml
- name: Start emulator
  uses: reactivecircus/android-emulator-runner@v2
  with:
    api-level: 34
    arch: x86_64
    script: swift test
```
