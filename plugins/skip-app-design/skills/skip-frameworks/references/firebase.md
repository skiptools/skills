# SkipFirebase Integration

## Available Modules

| Module | Purpose |
|--------|---------|
| SkipFirebaseCore | Firebase initialization |
| SkipFirebaseAuth | Authentication (email, Google, Apple, etc.) |
| SkipFirebaseFirestore | Cloud Firestore document database |
| SkipFirebaseStorage | Cloud Storage for files |
| SkipFirebaseDatabase | Realtime Database |
| SkipFirebaseFunctions | Cloud Functions |
| SkipFirebaseMessaging | Push notifications (FCM) |
| SkipFirebaseCrashlytics | Crash reporting |
| SkipFirebaseAnalytics | Event analytics |
| SkipFirebaseRemoteConfig | Remote configuration |
| SkipFirebaseAppCheck | App attestation |
| SkipFirebaseInstallations | Installation management |

## Setup

### 1. Package.swift

Add only the modules you need:

```swift
dependencies: [
    .package(url: "https://source.skip.dev/skip-firebase.git", "0.0.0"..<"2.0.0"),
],
targets: [
    .target(name: "MyApp", dependencies: [
        .product(name: "SkipFirebaseCore", package: "skip-firebase"),
        .product(name: "SkipFirebaseAuth", package: "skip-firebase"),
        .product(name: "SkipFirebaseFirestore", package: "skip-firebase"),
    ]),
]
```

### 2. Firebase Console

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Create a project (or use existing)
3. Add an iOS app with your bundle ID
4. Add an Android app with your package name
5. Download configuration files

### 3. Configuration Files

- **iOS**: Place `GoogleService-Info.plist` in `Darwin/`
- **Android**: Place `google-services.json` in `Android/app/`

### 4. Android Gradle

Add to `Android/app/build.gradle.kts`:

```kotlin
plugins {
    id("com.google.gms.google-services") version "4.4.4" apply true
    // If using Crashlytics:
    id("com.google.firebase.crashlytics") version "3.0.2" apply true
}
```

### 5. Initialize

```swift
import SkipFirebaseCore

// In your app's init or app delegate
FirebaseApp.configure()
```

## Authentication

```swift
import SkipFirebaseAuth

// Email sign-in
let result = try await Auth.auth().signIn(withEmail: email, password: password)
let user = result.user

// Sign out
try Auth.auth().signOut()

// Auth state listener
Auth.auth().addStateDidChangeListener { auth, user in
    if let user = user {
        print("Signed in: \(user.uid)")
    }
}

// Create account
try await Auth.auth().createUser(withEmail: email, password: password)
```

## Firestore

```swift
import SkipFirebaseFirestore

let db = Firestore.firestore()

// Read a document
let doc = try await db.collection("users").document("user1").getDocument()
let data = doc.data()

// Write a document
try await db.collection("users").document("user1").setData([
    "name": "Alice",
    "email": "alice@example.com"
])

// Query
let snapshot = try await db.collection("users")
    .whereField("age", isGreaterThan: 18)
    .order(by: "name")
    .limit(to: 10)
    .getDocuments()

for doc in snapshot.documents {
    print(doc.data())
}

// Real-time listener
let listener = db.collection("messages").addSnapshotListener { snapshot, error in
    guard let documents = snapshot?.documents else { return }
    // Update UI
}
// Later: listener.remove()
```

## Cloud Messaging (Push Notifications)

```swift
import SkipFirebaseMessaging

// Request permission
let settings = try await UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound])

// Get FCM token
let token = try await Messaging.messaging().token()

// Set delegate
Messaging.messaging().delegate = self
```

Android requires additional manifest entries:

```xml
<service android:name="skip.firebase.messaging.MessagingService" android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
<meta-data
    android:name="com.google.firebase.messaging.default_notification_channel_id"
    android:value="tools.skip.firebase.messaging" />
```

## Storage

```swift
import SkipFirebaseStorage

let storage = Storage.storage()
let ref = storage.reference().child("images/photo.jpg")

// Upload
let metadata = try await ref.putDataAsync(imageData)

// Download URL
let url = try await ref.downloadURL()

// Download data
let data = try await ref.dataAsync(maxSize: 10 * 1024 * 1024)
```
