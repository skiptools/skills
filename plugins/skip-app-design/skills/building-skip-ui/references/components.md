# SkipUI Component Support

## Layout Components

| Component | Support | Notes |
|-----------|---------|-------|
| `VStack` | Full | |
| `HStack` | Full | |
| `ZStack` | Full | |
| `Group` | Full | |
| `Spacer` | Full | |
| `GeometryReader` | Full | |
| `ScrollView` | Full | |
| `LazyVStack` | Full | |
| `LazyHStack` | Full | |
| `LazyVGrid` | Full | |
| `LazyHGrid` | Full | |
| `Grid` | Full | |
| `GridRow` | Full | |
| `Form` | Full | Renders as Material form on Android |
| `Section` | Full | |
| `List` | Full | |

## Control Components

| Component | Support | Notes |
|-----------|---------|-------|
| `Button` | Full | All standard button styles |
| `Text` | Full | Markdown rendering supported |
| `Label` | Full | |
| `TextField` | Full | |
| `SecureField` | Full | |
| `TextEditor` | Full | |
| `Toggle` | Full | |
| `Slider` | Full | |
| `Stepper` | Full | |
| `Picker` | Full | `.menu`, `.wheel`, `.segmented` styles |
| `DatePicker` | Medium | Date range constraints require Fuse bridging |
| `ProgressView` | Full | Linear and circular |
| `Link` | Full | |
| `Menu` | Full | |
| `ShareLink` | Full | |
| `DisclosureGroup` | Medium | Animation limitations |
| `ColorPicker` | Full | |

## Navigation Components

| Component | Support | Notes |
|-----------|---------|-------|
| `NavigationStack` | Full | |
| `NavigationLink` | Full | Value-based and destination-based |
| `NavigationSplitView` | Full | |
| `TabView` | Full | |
| `.navigationTitle` | Full | |
| `.navigationBarTitleDisplayMode` | Full | |
| `.toolbar` | Full | |
| `.searchable` | Full | |
| `.sheet` | Full | |
| `.fullScreenCover` | Full | |
| `.alert` | Full | |
| `.confirmationDialog` | Full | |

## Media and Graphics

| Component | Support | Notes |
|-----------|---------|-------|
| `Image` | Full | System images, asset catalog, named |
| `AsyncImage` | Full | |
| `Color` | Full | |
| `LinearGradient` | Full | |
| `EllipticalGradient` | Full | |
| `Circle` | Full | |
| `Capsule` | Full | |
| `Rectangle` | Full | |
| `RoundedRectangle` | Full | |
| `Path` | Full | |
| `Canvas` | Limited | Basic drawing operations only |
| `Map` | Full | |

## State Management

| Property Wrapper | Support | Notes |
|-----------------|---------|-------|
| `@State` | Full | |
| `@Binding` | Full | |
| `@StateObject` | Full | |
| `@ObservedObject` | Full | |
| `@EnvironmentObject` | Full | |
| `@Environment` | Full | |
| `@Bindable` | Full | Requires `@Observable` model |
| `@AppStorage` | Medium | Optional values not supported |
| `@Observable` | Full | Via SkipModel |
| `@Published` | Full | Via SkipModel |

## Common Modifiers (Full Support)

`.padding`, `.frame`, `.background`, `.foregroundStyle`, `.foregroundColor`, `.font`, `.fontWeight`, `.opacity`, `.cornerRadius`, `.clipShape`, `.shadow`, `.overlay`, `.border`, `.disabled`, `.hidden`, `.id`, `.tag`, `.onAppear`, `.onDisappear`, `.onTapGesture`, `.task`, `.animation`, `.transition`, `.offset`, `.rotationEffect`, `.scaleEffect`, `.sheet`, `.alert`, `.confirmationDialog`, `.toolbar`, `.navigationTitle`, `.navigationBarTitleDisplayMode`, `.searchable`, `.refreshable`, `.swipeActions`, `.contextMenu`, `.listStyle`, `.listRowBackground`, `.listRowSeparator`

## Not Available in SkipUI

- `UIViewRepresentable` / `UIViewControllerRepresentable`
- `matchedGeometryEffect`
- `@FetchRequest` (Core Data)
- `PhotosPicker`
- Custom `Shape` with complex `AnimatableData`
- `GraphicsContext` advanced operations
- `TimelineView`
- Widgets / App Intents
