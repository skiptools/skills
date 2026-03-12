# Data Storage Frameworks

## SkipSQL — SQLite Database

### Setup

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-sql.git", from: "1.0.0"),
.product(name: "SkipSQL", package: "skip-sql"),
```

No special `skip.yml` configuration needed.

### Usage

```swift
import SkipSQL

// Open or create database
let dbPath = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
    .appendingPathComponent("app.db").path
let db = try SQLContext(path: dbPath)

// Create table
try db.execute(sql: """
    CREATE TABLE IF NOT EXISTS items (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        quantity INTEGER DEFAULT 0,
        created_at REAL DEFAULT (julianday('now'))
    )
""")

// Insert
try db.execute(sql: "INSERT INTO items (name, quantity) VALUES (?, ?)",
    parameters: [.text("Widget"), .integer(10)])

// Query
let rows = try db.query(sql: "SELECT id, name, quantity FROM items WHERE quantity > ?",
    parameters: [.integer(5)])
for row in rows {
    let id = row[0]?.integerValue
    let name = row[1]?.textValue
    let qty = row[2]?.integerValue
}

// Update
try db.execute(sql: "UPDATE items SET quantity = ? WHERE id = ?",
    parameters: [.integer(20), .integer(1)])

// Delete
try db.execute(sql: "DELETE FROM items WHERE id = ?",
    parameters: [.integer(1)])
```

### Schema Migrations

```swift
try db.migrate(migrations: [
    // Migration 1: initial schema
    { db in
        try db.execute(sql: "CREATE TABLE items (id INTEGER PRIMARY KEY, name TEXT)")
    },
    // Migration 2: add column
    { db in
        try db.execute(sql: "ALTER TABLE items ADD COLUMN quantity INTEGER DEFAULT 0")
    },
])
```

### Notes

- SQLite version varies: 3.22 (Android API 28) through 3.50 (API 36)
- No built-in thread safety — use external locking for concurrent access
- For encryption and FTS, use `SkipSQLPlus` (adds ~1MB)

## SkipKeychain — Secure Key/Value Storage

### Setup

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-keychain.git", "0.0.0"..<"2.0.0"),
.product(name: "SkipKeychain", package: "skip-keychain"),
```

```yaml
# skip.yml
skip:
  bridging: true

build:
  contents:
    - block: 'dependencies'
      contents:
        - 'implementation("androidx.security:security-crypto:1.0.0")'
```

### Usage

```swift
import SkipKeychain

// Store
Keychain.shared.set("my-secret-token", forKey: "auth_token")

// Retrieve
if let token = Keychain.shared.string(forKey: "auth_token") {
    print("Token: \(token)")
}

// Remove
Keychain.shared.removeValue(forKey: "auth_token")
```

### Platform Details

- **iOS**: Uses the system Keychain (persists across app reinstalls)
- **Android**: Uses `EncryptedSharedPreferences` (AES-256, cleared on app uninstall)
- Store only string values — serialize complex data to JSON first

## UserDefaults (via SkipFoundation)

For non-sensitive preferences, `UserDefaults` is available without additional dependencies:

```swift
UserDefaults.standard.set("dark", forKey: "theme")
let theme = UserDefaults.standard.string(forKey: "theme")
UserDefaults.standard.set(42, forKey: "highScore")
```

In SwiftUI, use `@AppStorage`:

```swift
@AppStorage("theme") var theme = "light"
// Note: optional values in @AppStorage are not supported in SkipUI
```
