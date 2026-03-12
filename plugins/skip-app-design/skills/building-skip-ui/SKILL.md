---
name: building-skip-ui
description: Building SwiftUI views in Skip projects that work on both iOS and Android. Covers supported components, navigation, state management, Jetpack Compose customization, Material 3 theming, and cross-platform UI patterns.
---

# Building Skip UI

SkipUI converts SwiftUI into Jetpack Compose for Android. Use standard SwiftUI patterns — SkipUI handles the translation. Not all SwiftUI APIs are available; this skill covers what works and how to customize the Android rendering.

## Project Setup

A Skip app uses standard SwiftUI with the SkipUI framework:

```swift
// Package.swift dependencies
.package(url: "https://source.skip.dev/skip.git", from: "1.2.0"),
.package(url: "https://source.skip.dev/skip-ui.git", from: "1.0.0"),
.package(url: "https://source.skip.dev/skip-foundation.git", from: "1.0.0"),
```

Create projects with:
```bash
skip init --appid=com.example.myapp my-app MyApp          # Lite
skip init --native-app --appid=com.example.myapp my-app MyApp  # Fuse
```

## Supported Components

Consult the reference files for detailed component tables:
- ./references/components.md -- Full component support table
- ./references/compose-customization.md -- Compose and Material 3 customization
- ./references/patterns.md -- Common cross-platform UI patterns

### Layout (Full Support)
`VStack`, `HStack`, `ZStack`, `Group`, `Spacer`, `GeometryReader`, `ScrollView`, `LazyVStack`, `LazyHStack`, `LazyVGrid`, `LazyHGrid`, `Grid`, `GridRow`, `Form`, `Section`

### Controls (Full Support)
`Button`, `Text`, `Label`, `TextField`, `SecureField`, `TextEditor`, `Toggle`, `Slider`, `Stepper`, `Picker`, `ProgressView`, `Link`, `Menu`, `ShareLink`

### Navigation (Full Support)
`NavigationStack`, `NavigationLink`, `NavigationSplitView`, `TabView`, `.sheet`, `.fullScreenCover`, `.alert`, `.confirmationDialog`, `.toolbar`, `.searchable`

### Media (Full Support)
`Image`, `AsyncImage`, `Color`, `LinearGradient`, `Circle`, `Capsule`, `Rectangle`, `RoundedRectangle`, `Path`

### State Management (Full Support)
`@State`, `@Binding`, `@StateObject`, `@ObservedObject`, `@EnvironmentObject`, `@Environment`, `@Bindable`, `@Observable` (via SkipModel)

### Limited or Unavailable
- `DatePicker`: Medium support (date range constraints need Fuse bridging)
- `@AppStorage`: Optional values not supported
- `Canvas`: Basic drawing only
- `UIViewRepresentable`: Not available — use `#if os(iOS)` guards
- `matchedGeometryEffect`: Not available
- `@FetchRequest`: Not available — use SkipSQL or SkipFirebase
- Custom `Shape` with complex `AnimatableData`: Not available

## Cross-Platform View Pattern

```swift
import SwiftUI

struct ContentView: View {
    @State private var count = 0

    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                Text("Count: \(count)")
                    .font(.title)
                Button("Increment") {
                    count += 1
                }
                .buttonStyle(.borderedProminent)
            }
            .navigationTitle("My App")
        }
    }
}
```

## Observable Models

```swift
import Foundation
import Observation

@Observable final class ViewModel {
    var items: [String] = []
    var isLoading = false

    func loadItems() async {
        isLoading = true
        defer { isLoading = false }
        // Fetch data...
    }
}
```

## Platform-Specific Code

```swift
var body: some View {
    Text("Hello")
        #if os(Android)
        .font(.system(size: 18))
        #endif
}
```

## Compose Customization

Embed raw Jetpack Compose in `#if SKIP` blocks:

```swift
#if SKIP
ComposeView { context in
    androidx.compose.material3.FloatingActionButton(
        onClick: { handleTap() }
    ) {
        androidx.compose.material3.Icon(
            imageVector: androidx.compose.material.icons.Icons.Default.Add,
            contentDescription: "Add"
        )
    }
}
#endif
```

Apply Compose modifiers to SwiftUI views:

```swift
Text("Custom")
    #if SKIP
    .composeModifier { modifier in modifier.testTag("customText") }
    #endif
```

Customize Material 3 theming:

```swift
ContentView()
    .material3ColorScheme(ColorScheme(primary: Color.blue, onPrimary: Color.white))
```

## References

Consult these resources for detailed guidance:
- ./references/components.md -- Complete component and modifier support tables
- ./references/compose-customization.md -- ComposeView, composeModifier, Material 3
- ./references/patterns.md -- Tab navigation, lists, async loading, forms
