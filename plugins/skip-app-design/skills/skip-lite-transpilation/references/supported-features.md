# Supported and Unsupported Swift Features in Skip Lite

## Fully Supported

- Classes, structs, enums (including associated values and raw values)
- Protocols and protocol extensions
- Extensions (with limitations on external types)
- Generics (basic type parameters and constraints)
- Closures and higher-order functions
- Optional chaining (`?.`) and nil coalescing (`??`)
- `async`/`await` (transpiled to Kotlin suspend functions and coroutines)
- `@Observable` and `@Published` (via SkipModel → Compose `MutableState`)
- Pattern matching (`switch`/`case` with value binding)
- `guard`, `if let`, `while let`
- `defer` statements
- `Codable` / `Encodable` / `Decodable`
- String interpolation
- Collection operations: `map`, `filter`, `reduce`, `compactMap`, `flatMap`, `sorted`, `contains`, `first`, `last`
- `for`/`in` loops with `where` clauses
- Nested types
- Access control (`public`, `internal`, `private`)
- Type aliases (basic)
- Computed properties and property observers (`willSet`, `didSet`)
- Subscripts
- `@MainActor` (transpiled to main thread dispatch)
- `throws`/`try`/`catch`
- `lazy` properties
- Tuple types (basic)
- Numeric literals and type conversions

## Partially Supported

| Feature | Limitation |
|---------|-----------|
| Key paths | Cannot be used as values; property access syntax works |
| Existential types (`any P`) | Limited; prefer concrete types or generics |
| `Result` type | Supported but prefer `throws` patterns |
| Type aliases with generics | Constrained generic type aliases not supported |
| Extensions on external types | Cannot add protocols or initializers to types from other modules |
| Nested generics | Types nested within generic types have incompatible behavior |

## Not Supported

| Feature | Alternative |
|---------|-------------|
| C interop / `UnsafePointer` | Use SkipFFI with JNA |
| Runtime reflection / `Mirror` | Use `Codable` for introspection |
| `#selector` / Objective-C runtime | Use closures |
| Variadic generics | Use overloads or arrays |
| Parameter packs | Use overloads |
| Custom result builders | Only `@ViewBuilder` (via SkipUI) |
| Dynamic member lookup | Use explicit property access |
| Complex macros | Only Skip-supported macros (`@Observable`, `@Test`, etc.) |
| `UIViewRepresentable` | Use `#if os(iOS)` guard + `ComposeView` on Android |
| `@propertyWrapper` (custom) | Limited; use Skip-provided wrappers |
| String mutation (`.append`) | Use `+=` or string concatenation |
| Local type declarations | Move types to module scope |
| OptionSet as non-struct | Must be a `struct` |

## Foundation API Support (via SkipFoundation)

### Full Support
`JSONEncoder`, `JSONDecoder`, `JSONSerialization`, `Locale`, `TimeZone`, `HTTPURLResponse`, `URLResponse`, `NSError`, `NSLock`, `Notification`, `NotificationCenter`, `OSLog.Logger`, `DateInterval`, `ComparisonResult`, `LocalizedStringResource`

### High Support
`FileManager`, `Data`, `DateComponents`, `Calendar`, `DateFormatter`, `ISO8601DateFormatter`, `Bundle`, `URL`, `URLSession`, `URLRequest`, `UserDefaults`, `ProcessInfo`, `UUID`, `NumberFormatter`

### Medium Support
`CharacterSet`, `Decimal` (aliased to `java.math.BigDecimal`), `DispatchQueue` (basic async only), `AttributedString` (Markdown/localization limited)

### Not Available
`NSPredicate`, `NSRegularExpression` (use Swift `Regex`), `NSCache`, `NSKeyedArchiver`/`NSKeyedUnarchiver`, Core Data, CloudKit, HealthKit, StoreKit, and other Apple-only frameworks
