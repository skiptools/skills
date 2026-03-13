---
name: skip-frameworks
description: Integrating optional Skip frameworks into a project. Covers Firebase, SQLite, Keychain, WebView, audio/video, device sensors, Lottie animations, FFI, and other frameworks. Includes Package.swift dependencies, skip.yml configuration, and usage examples.
---

# Skip Frameworks

Use this skill when adding optional frameworks to a Skip project.

## Adding a Framework

1. Add package dependency to `Package.swift`
2. Add product to target dependencies
3. Configure `Skip/skip.yml` if Android-specific Gradle dependencies are needed
4. Import the module in Swift code

## Framework Reference

Consult the detailed reference files:
- ./references/firebase.md -- Firebase setup, auth, Firestore, messaging, crashlytics
- ./references/data-storage.md -- SkipSQL, SkipKeychain, UserDefaults
- ./references/media-web.md -- SkipWeb, SkipAV, SkipMotion

## Quick Reference

### SkipSQL — SQLite

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-sql.git", from: "1.0.0"),
.product(name: "SkipSQL", package: "skip-sql"),
```

```swift
import SkipSQL
let db = try SQLContext(path: dbPath)
try db.execute(sql: "CREATE TABLE items (id INTEGER PRIMARY KEY, name TEXT)")
try db.execute(sql: "INSERT INTO items (name) VALUES (?)", parameters: [.text("Widget")])
```

### SkipKeychain — Secure Storage

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-keychain.git", "0.0.0"..<"2.0.0"),
.product(name: "SkipKeychain", package: "skip-keychain"),
```

```swift
import SkipKeychain
Keychain.shared.set("secret", forKey: "token")
let token = Keychain.shared.string(forKey: "token")
```

### SkipFirebase — Firebase

```swift
// Package.swift (add only what you need)
.package(url: "https://source.skip.dev/skip-firebase.git", "0.0.0"..<"2.0.0"),
.product(name: "SkipFirebaseAuth", package: "skip-firebase"),
.product(name: "SkipFirebaseFirestore", package: "skip-firebase"),
```

Setup: `GoogleService-Info.plist` in `Darwin/`, `google-services.json` in `Android/app/`.

### SkipWeb — WebView

```swift
.package(url: "https://source.skip.dev/skip-web.git", from: "1.0.0"),
.product(name: "SkipWeb", package: "skip-web"),
```

### SkipAV — Audio/Video

```swift
.package(url: "https://source.skip.dev/skip-av.git", from: "1.0.0"),
.product(name: "SkipAV", package: "skip-av"),
```

### SkipDevice — Location, Sensors, Network

```swift
.package(url: "https://source.skip.dev/skip-device.git", "0.0.0"..<"2.0.0"),
.product(name: "SkipDevice", package: "skip-device"),
```

### Other Frameworks

| Framework | Package | Purpose |
|-----------|---------|---------|
| SkipMotion | `skip-motion` | Lottie animations |
| SkipFFI | `skip-ffi` | C/C++ interop via JNA |
| SkipBluetooth | `skip-bluetooth` | Bluetooth LE |
| SkipNFC | `skip-nfc` | NFC reading/writing |
| SkipQRCode | `skip-qrcode` | QR code generation/scanning |
| SkipXML | `skip-xml` | XML parsing |
| SkipZip | `skip-zip` | ZIP compression |
| SkipScript | `skip-script` | JavaScript engine |
| SkipSupabase | `skip-supabase` | Supabase client |
| SkipStripe | `skip-stripe` | Stripe payments |
| SkipSentry | `skip-sentry` | Error tracking |
| SkipPostHog | `skip-posthog` | Analytics |
