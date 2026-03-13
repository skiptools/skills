# Compose Customization

SkipUI provides escape hatches for fine-grained control over the Android Jetpack Compose rendering. All Compose customization should be wrapped in `#if SKIP` blocks so it compiles only on Android.

## ComposeView

Embed raw Jetpack Compose UI directly in your SwiftUI view hierarchy:

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

## composeModifier

Apply Compose modifiers to any SwiftUI view:

```swift
Text("Custom styled")
    #if SKIP
    .composeModifier { modifier in
        modifier
            .testTag("myText")
            .semantics { contentDescription = "Custom text" }
    }
    #endif
```

## Compose(context:) Method

Use SwiftUI views within Compose code:

```swift
#if SKIP
ComposeView { context in
    androidx.compose.foundation.layout.Column {
        // Use a SwiftUI view inside Compose
        Text("SwiftUI inside Compose").Compose(context: context)
    }
}
#endif
```

## Material 3 Color Scheme

Customize the Material 3 color scheme for the entire app or a subtree:

```swift
ContentView()
    .material3ColorScheme(ColorScheme(
        primary: Color.blue,
        onPrimary: Color.white,
        secondary: Color.green,
        surface: Color(.systemBackground)
    ))
```

## Material 3 Component Modifiers

Customize individual Material 3 components:

```swift
// Navigation bar
TabView { ... }
    .material3NavigationBar { config in
        config.containerColor = Color.blue
    }

// Text field
TextField("Search", text: $query)
    .material3TextField { config in
        config.colors = TextFieldDefaults.colors(
            focusedIndicatorColor: .clear
        )
    }

// Button
Button("Action") { ... }
    .material3Button { config in
        config.shape = RoundedCornerShape(8.dp)
    }

// Top app bar
NavigationStack { ... }
    .material3TopAppBar { config in
        config.colors = TopAppBarDefaults.topAppBarColors(
            containerColor: Color.blue
        )
    }
```

Available Material 3 modifiers:
- `material3NavigationBar`
- `material3TextField`
- `material3Button`
- `material3TopAppBar`
- `material3BottomSheet`

## LazyItemScope Animations

```swift
List(items) { item in
    Text(item.name)
        #if SKIP
        .composeModifier { modifier in
            LazyItemScope.animateItem(modifier)
        }
        #endif
}
```
