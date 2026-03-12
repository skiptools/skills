---
name: skip-project-creation
description: Creating new Skip app and library projects. Handles skip init commands for both Skip Lite (transpiled) and Skip Fuse (native) modes, adding optional framework dependencies to Package.swift, and scaffolding template code for frameworks like SkipSQL, SkipFirebase, SkipKeychain, SkipWeb, and others.
---

# Skip Project Creation

Create new Skip apps and libraries using `skip init`, then optionally add framework dependencies with template code.

## Prerequisites

```bash
brew install skiptools/skip/skip
skip checkup
```

## Creating a New App

### Skip Lite (Transpiled) App

Swift is transpiled to Kotlin. Best for most projects.

```bash
skip init --transpiled-app --appid=com.example.myapp my-app MyApp MyAppModel
```

- `--transpiled-app` — Skip Lite mode (Swift transpiled to Kotlin)
- `--appid=BUNDLE_ID` — Required bundle identifier
- `my-app` — Project folder name (lowercase, hyphens OK)
- `MyApp` — UI module name (CamelCase)
- `MyAppModel` — Model module name (optional second module for separation)

### Skip Fuse (Native) App

Swift compiles natively on Android. Full Swift language support.

```bash
skip init --native-app --appid=com.example.someapp some-app SomeApp
```

- `--native-app` — Skip Fuse mode (native Swift compilation)
- Fuse apps typically use a single module (no separate model module)

### Common Options

| Flag | Purpose |
|------|---------|
| `--appid=ID` | Bundle identifier (required for apps) |
| `--version=VER` | Initial version (default: 0.0.1) |
| `--no-icon` | Skip icon generation |
| `--icon=PATH` | Custom icon source (SVG, PDF, PNG) |
| `--open-xcode` | Open Xcode after creation |
| `--no-build` | Skip the initial build |
| `--git-repo` | Initialize a git repository |

### Creating Libraries

```bash
# Transpiled (Lite) library
skip init --transpiled-model my-lib MyLib

# Native (Fuse) library
skip init --native-model my-lib MyLib
```

## Adding Dependencies After Creation

After `skip init` creates the project, add optional framework dependencies by modifying `Package.swift` and adding template code.

### Step-by-step

1. **Add the package dependency** to the top-level `dependencies` array in `Package.swift`
2. **Add the product** to the appropriate target's `dependencies` array
3. **Configure `skip.yml`** if the framework needs Android Gradle dependencies
4. **Add template code** to the appropriate source file (UI module or Model module)

### Where to Add Code

- **UI code** (views, navigation): `Sources/<AppModule>/ContentView.swift`
- **Model/data code** (storage, network, auth): `Sources/<ModelModule>/ViewModel.swift`
- **If single module**: Both go in the app module's source files

## Dependency Templates

Consult the detailed dependency templates reference:
- ./references/dependency-templates.md -- Package.swift entries, skip.yml config, and starter code for each framework

### Quick Reference: Package.swift Dependency Syntax

Stable frameworks use `from:` versioning:
```swift
.package(url: "https://source.skip.dev/skip-sql.git", from: "1.0.0"),
```

Pre-stable frameworks use range versioning:
```swift
.package(url: "https://source.skip.dev/skip-firebase.git", "0.0.0"..<"2.0.0"),
```

Product dependencies go in the target that uses them:
```swift
.target(name: "MyApp", dependencies: [
    "MyAppModel",
    .product(name: "SkipUI", package: "skip-ui"),
    .product(name: "SkipSQL", package: "skip-sql"),  // added
], ...),
```

## Example Workflows

### "Create a Skip Lite app called MyApp with SQLite"

```bash
skip init --transpiled-app --appid=com.example.myapp my-app MyApp MyAppModel --no-build
```

Then add SkipSQL to `Package.swift` in the model target's dependencies, and add the database helper code to `Sources/MyAppModel/ViewModel.swift`.

### "Create a Skip Fuse app called ChatApp with Firebase"

```bash
skip init --native-app --appid=com.example.chatapp chat-app ChatApp --no-build
```

Then add SkipFirebase packages to `Package.swift`, place config files, and add initialization code.

### "Create a Skip Lite app with Keychain and Web"

```bash
skip init --transpiled-app --appid=com.example.demo demo-app DemoApp DemoAppModel --no-build
```

Then add SkipKeychain and SkipWeb to `Package.swift` and add template code to the appropriate modules.
