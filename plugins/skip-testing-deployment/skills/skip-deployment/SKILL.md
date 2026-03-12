---
name: skip-deployment
description: Exporting, signing, and distributing Skip apps. Covers skip export command, iOS archive and App Store submission, Android APK/AAB signing, ProGuard, app icons, Skip.env metadata, and CI/CD with GitHub Actions.
---

# Skip Deployment

Build, sign, and distribute Skip apps for the App Store and Google Play.

## Building for Distribution

```bash
skip export                     # Both platforms, debug
skip export -c release          # Both platforms, release
skip export --android           # Android only
skip export --ios               # iOS only
skip export -c release -o ./out # Custom output directory
```

## App Metadata

### Skip.env

```properties
SKIP_APP_NAME=MyApp
SKIP_APP_ID=com.example.myapp
SKIP_APP_VERSION=1.0.0
SKIP_APP_BUILD=1
```

### App Icons

```bash
skip icon --source icon-1024.png
```

## iOS Distribution

Archive via Xcode: **Product > Archive**, then **Distribute App** in Organizer.

Or via command line:
```bash
xcodebuild archive -workspace Project.xcworkspace -scheme MyApp -configuration Release -archivePath ./build/MyApp.xcarchive
xcodebuild -exportArchive -archivePath ./build/MyApp.xcarchive -exportOptionsPlist ExportOptions.plist -exportPath ./build/
```

## Android Distribution

### Signing

Configure in `Android/app/build.gradle.kts`:

```kotlin
android {
    signingConfigs {
        create("release") {
            storeFile = file("keystore.jks")
            storePassword = System.getenv("KEYSTORE_PASSWORD") ?: ""
            keyAlias = System.getenv("KEY_ALIAS") ?: ""
            keyPassword = System.getenv("KEY_PASSWORD") ?: ""
        }
    }
    buildTypes {
        getByName("release") {
            signingConfig = signingConfigs.getByName("release")
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }
}
```

### Create Keystore

```bash
keytool -genkey -v -keystore keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias myapp
```

Never commit the keystore to version control.

### Build AAB for Play Store

```bash
cd Android && ./gradlew :app:bundleRelease
# Output: Android/app/build/outputs/bundle/release/app-release.aab
```

## CI/CD

Consult the reference for a complete GitHub Actions workflow:
- ./references/cicd.md -- GitHub Actions workflow, environment setup, artifact upload

### Minimal GitHub Actions

```yaml
name: Build and Test
on: [push, pull_request]
jobs:
  build:
    runs-on: macos-14
    steps:
      - uses: actions/checkout@v4
      - name: Install Skip
        run: brew install skiptools/skip/skip
      - name: Test
        run: swift test
      - name: Export
        run: skip export -c release -o ./artifacts
      - uses: actions/upload-artifact@v4
        with:
          name: app-artifacts
          path: ./artifacts/
```
