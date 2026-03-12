# Dependency Templates

Complete templates for adding optional frameworks to a Skip project after creation. Each section includes the Package.swift entries, any skip.yml configuration, and starter code.

## SkipSQL — SQLite Database

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-sql.git", from: "1.0.0"),

// Add to model target dependencies:
.product(name: "SkipSQL", package: "skip-sql"),
```

No skip.yml changes needed.

### Template Code (Model module — ViewModel.swift)

```swift
import SkipSQL

/// Database manager for the app
class DatabaseManager {
    static let shared = DatabaseManager()
    private let db: SQLContext

    init() {
        let dbPath = URL.applicationSupportDirectory
            .appendingPathComponent("app.db").path
        do {
            try FileManager.default.createDirectory(
                at: URL.applicationSupportDirectory,
                withIntermediateDirectories: true
            )
            db = try SQLContext(path: dbPath)
            try createTables()
        } catch {
            fatalError("Failed to open database: \(error)")
        }
    }

    private func createTables() throws {
        try db.execute(sql: """
            CREATE TABLE IF NOT EXISTS items (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                created_at REAL DEFAULT (julianday('now'))
            )
        """)
    }

    func addItem(name: String) throws {
        try db.execute(sql: "INSERT INTO items (name) VALUES (?)",
            parameters: [.text(name)])
    }

    func allItems() throws -> [(id: Int, name: String)] {
        let rows = try db.query(sql: "SELECT id, name FROM items ORDER BY created_at DESC")
        return rows.compactMap { row in
            guard let id = row[0]?.integerValue, let name = row[1]?.textValue else { return nil }
            return (id: id, name: name)
        }
    }

    func deleteItem(id: Int) throws {
        try db.execute(sql: "DELETE FROM items WHERE id = ?",
            parameters: [.integer(id)])
    }
}
```

## SkipKeychain — Secure Storage

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-keychain.git", "0.0.0"..<"2.0.0"),

// Add to model target dependencies:
.product(name: "SkipKeychain", package: "skip-keychain"),
```

### skip.yml (model module)

```yaml
skip:
  mode: 'transpiled'
  bridging: true

build:
  contents:
    - block: 'dependencies'
      contents:
        - 'implementation("androidx.security:security-crypto:1.0.0")'
```

### Template Code (Model module)

```swift
import SkipKeychain

extension ViewModel {
    /// Store a value securely
    func setSecure(_ value: String, forKey key: String) {
        Keychain.shared.set(value, forKey: key)
    }

    /// Retrieve a secure value
    func getSecure(forKey key: String) -> String? {
        Keychain.shared.string(forKey: key)
    }

    /// Remove a secure value
    func removeSecure(forKey key: String) {
        Keychain.shared.removeValue(forKey: key)
    }
}
```

## SkipFirebaseAuth — Firebase Authentication

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-firebase.git", "0.0.0"..<"2.0.0"),

// Add to target dependencies (add only modules you need):
.product(name: "SkipFirebaseCore", package: "skip-firebase"),
.product(name: "SkipFirebaseAuth", package: "skip-firebase"),
```

### Setup Files

- Place `GoogleService-Info.plist` in `Darwin/`
- Place `google-services.json` in `Android/app/`

### Android Gradle (Android/app/build.gradle.kts)

Add to plugins block:
```kotlin
id("com.google.gms.google-services") version "4.4.4" apply true
```

### Template Code (Model module)

```swift
import SkipFirebaseCore
import SkipFirebaseAuth

@Observable class AuthManager {
    var currentUser: User?
    var isSignedIn: Bool { currentUser != nil }

    init() {
        FirebaseApp.configure()
        Auth.auth().addStateDidChangeListener { _, user in
            self.currentUser = user
        }
    }

    func signIn(email: String, password: String) async throws {
        let result = try await Auth.auth().signIn(withEmail: email, password: password)
        currentUser = result.user
    }

    func createAccount(email: String, password: String) async throws {
        let result = try await Auth.auth().createUser(withEmail: email, password: password)
        currentUser = result.user
    }

    func signOut() throws {
        try Auth.auth().signOut()
        currentUser = nil
    }
}
```

## SkipFirebaseFirestore — Cloud Firestore

### Package.swift

```swift
// Add to dependencies (if not already added for Auth):
.package(url: "https://source.skip.dev/skip-firebase.git", "0.0.0"..<"2.0.0"),

// Add to target dependencies:
.product(name: "SkipFirebaseCore", package: "skip-firebase"),
.product(name: "SkipFirebaseFirestore", package: "skip-firebase"),
```

### Template Code (Model module)

```swift
import SkipFirebaseCore
import SkipFirebaseFirestore

@Observable class FirestoreManager {
    var documents: [[String: Any]] = []
    private let db = Firestore.firestore()

    func fetchCollection(_ name: String) async throws {
        let snapshot = try await db.collection(name).getDocuments()
        documents = snapshot.documents.map { $0.data() }
    }

    func addDocument(to collection: String, data: [String: Any]) async throws {
        try await db.collection(collection).addDocument(data: data)
    }

    func deleteDocument(in collection: String, id: String) async throws {
        try await db.collection(collection).document(id).delete()
    }
}
```

## SkipWeb — WebView

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-web.git", from: "1.0.0"),

// Add to UI target dependencies:
.product(name: "SkipWeb", package: "skip-web"),
```

### skip.yml (UI module)

```yaml
skip:
  mode: 'transpiled'
  bridging: true

build:
  contents:
    - block: 'dependencies'
      export: false
      contents:
        - 'implementation("androidx.browser:browser:1.9.0")'
        - 'implementation("androidx.webkit:webkit:1.15.0")'
```

### Template Code (UI module — ContentView.swift)

```swift
import SkipWeb

struct BrowserView: View {
    let url: URL
    @State private var navigator = WebViewNavigator()

    var body: some View {
        WebView(url: url, navigator: navigator)
            .navigationTitle("Browser")
            .toolbar {
                ToolbarItem(placement: .bottomBar) {
                    HStack {
                        Button("Back") { navigator.goBack() }
                        Spacer()
                        Button("Forward") { navigator.goForward() }
                        Spacer()
                        Button("Reload") { navigator.reload() }
                    }
                }
            }
    }
}
```

## SkipAV — Audio and Video

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-av.git", from: "1.0.0"),

// Add to UI target dependencies:
.product(name: "SkipAV", package: "skip-av"),
```

Check the [skip-av repository](https://github.com/skiptools/skip-av) for the current skip.yml Media3 dependencies.

### Template Code (UI module)

```swift
import AVFoundation
import AVKit

struct VideoPlayerView: View {
    let url: URL
    @State private var player: AVPlayer?

    var body: some View {
        VideoPlayer(player: player ?? AVPlayer())
            .onAppear {
                player = AVPlayer(url: url)
                player?.play()
            }
            .onDisappear {
                player?.pause()
            }
    }
}
```

## SkipDevice — Location, Sensors, Network

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-device.git", "0.0.0"..<"2.0.0"),

// Add to model target dependencies:
.product(name: "SkipDevice", package: "skip-device"),
```

### skip.yml (model module)

```yaml
skip:
  bridging: true
```

### Permissions

**iOS** (xcconfig): Add `NSLocationWhenInUseUsageDescription`

**Android** (AndroidManifest.xml):
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

### Template Code (Model module)

```swift
import SkipDevice

@Observable class LocationManager {
    var latitude: Double = 0.0
    var longitude: Double = 0.0
    var isTracking = false

    func requestCurrentLocation() async throws {
        let location = try await LocationProvider.shared.currentLocation()
        latitude = location.latitude
        longitude = location.longitude
    }

    func startTracking() {
        isTracking = true
        Task {
            for try await location in LocationProvider.shared.locationUpdates() {
                latitude = location.latitude
                longitude = location.longitude
            }
        }
    }
}
```

## SkipMotion — Lottie Animations

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-motion.git", "0.0.0"..<"2.0.0"),

// Add to UI target dependencies:
.product(name: "SkipMotion", package: "skip-motion"),
```

### skip.yml (UI module)

```yaml
skip:
  mode: 'transpiled'
  bridging: true

settings:
  contents:
    - block: 'dependencyResolutionManagement'
      contents:
        - block: 'versionCatalogs'
          contents:
            - block: 'create("libs")'
              contents:
                - 'version("lottie-compose", "6.7.1")'
                - 'library("lottie-compose", "com.airbnb.android", "lottie-compose").versionRef("lottie-compose")'

build:
  contents:
    - block: 'dependencies'
      export: false
      contents:
        - 'implementation(libs.lottie.compose)'
```

### Template Code (UI module)

```swift
import SkipMotion

struct AnimationView: View {
    var body: some View {
        if let url = Bundle.module.url(forResource: "animation", withExtension: "json"),
           let data = try? Data(contentsOf: url) {
            MotionView(lottie: data)
        }
    }
}
```

Place Lottie JSON files in `Sources/<Module>/Resources/`.

## SkipSupabase — Supabase Client

### Package.swift

```swift
// Add to dependencies:
.package(url: "https://source.skip.dev/skip-supabase.git", "0.0.0"..<"2.0.0"),

// Add to model target dependencies:
.product(name: "SkipSupabase", package: "skip-supabase"),
```

### Template Code (Model module)

```swift
import SkipSupabase

let supabase = SupabaseClient(
    supabaseURL: URL(string: "https://YOUR_PROJECT.supabase.co")!,
    supabaseKey: "YOUR_ANON_KEY"
)

// Query example:
// let items: [Item] = try await supabase.from("items").select().execute().value
```

## Multiple Dependencies Example

A project using SkipSQL and SkipKeychain together:

### Package.swift

```swift
let package = Package(
    name: "my-app",
    defaultLocalization: "en",
    platforms: [.iOS(.v17), .macOS(.v14)],
    products: [
        .library(name: "MyAppModule", type: .dynamic, targets: ["MyApp"])
    ],
    dependencies: [
        .package(url: "https://source.skip.tools/skip.git", from: "1.0.0"),
        .package(url: "https://source.skip.tools/skip-ui.git", from: "1.0.0"),
        .package(url: "https://source.skip.tools/skip-model.git", from: "1.0.0"),
        .package(url: "https://source.skip.tools/skip-foundation.git", from: "1.0.0"),
        // Added frameworks:
        .package(url: "https://source.skip.dev/skip-sql.git", from: "1.0.0"),
        .package(url: "https://source.skip.dev/skip-keychain.git", "0.0.0"..<"2.0.0"),
    ],
    targets: [
        .target(name: "MyApp", dependencies: [
            "MyAppModel",
            .product(name: "SkipUI", package: "skip-ui"),
        ], resources: [.process("Resources")], plugins: [.plugin(name: "skipstone", package: "skip")]),
        .target(name: "MyAppModel", dependencies: [
            .product(name: "SkipFoundation", package: "skip-foundation"),
            .product(name: "SkipModel", package: "skip-model"),
            .product(name: "SkipSQL", package: "skip-sql"),
            .product(name: "SkipKeychain", package: "skip-keychain"),
        ], resources: [.process("Resources")], plugins: [.plugin(name: "skipstone", package: "skip")]),
        .testTarget(name: "MyAppTests", dependencies: [
            "MyApp",
            .product(name: "SkipTest", package: "skip"),
        ], resources: [.process("Resources")], plugins: [.plugin(name: "skipstone", package: "skip")]),
    ]
)
```

## Framework → Target Placement Guide

| Framework | Target | Reason |
|-----------|--------|--------|
| SkipSQL | Model | Data persistence layer |
| SkipKeychain | Model | Secure credential storage |
| SkipFirebaseCore | Model | Firebase initialization |
| SkipFirebaseAuth | Model | Authentication logic |
| SkipFirebaseFirestore | Model | Data access |
| SkipFirebaseMessaging | Model | Push notification handling |
| SkipWeb | UI | WebView is a visual component |
| SkipAV | UI | Video/audio players are visual |
| SkipMotion | UI | Lottie animations are visual |
| SkipDevice | Model | Location/sensor data feeds model |
| SkipSupabase | Model | Backend data access |
| SkipFFI | Model | C/C++ interop for data processing |
