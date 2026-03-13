---
name: skip-lite-transpilation
description: Critical transpilation rules, patterns, and pitfalls for Skip Lite mode where Swift is transpiled to Kotlin. Covers integer overflow, type comparison errors, unsupported Swift features, and Kotlin interop. Use when writing or reviewing Swift code in a transpiled Skip project.
---

# Skip Lite Transpilation Rules

**CRITICAL**: `swift build` does NOT validate Kotlin transpilation. You MUST run `swift test` to catch transpilation errors.

## Impact: CRITICAL — Silent Data Corruption

### Integer Overflow

Swift `Int` is 64-bit. Kotlin `Int` is 32-bit (max ~2.1 billion). Intermediate arithmetic silently overflows in Kotlin.

```swift
// BAD: silently wrong on Android
let product = a * b  // if a * b > 2,147,483,647

// GOOD: use Int64 for large values
let product = Int64(a) * Int64(b)
```

## Impact: HIGH — Kotlin Compile Errors

### Double-to-Int Comparison

```swift
// BAD — Kotlin compile error
if someDouble == 0 { }

// GOOD
if someDouble == 0.0 { }
```

### Foundation-Qualified Math

```swift
// BAD — Kotlin compile error
let x = Foundation.sqrt(2.0)

// GOOD
let x = sqrt(2.0)
```

### String.append(Character)

```swift
// BAD — Kotlin compile error
result.append(someCharacter)

// GOOD
result += String(someCharacter)
```

## Type Guidance

- Use `final class` over `struct` for complex model types — struct value-semantics transpilation can be subtle
- Prefer named methods (`add()`, `multiply()`) over operator overloads for non-trivial types
- `static let` works well for constants and singletons

## Supported Features

Closures, generics (basic), optionals, `async`/`await`, `@Observable`, pattern matching, `guard`/`if let`, `defer`, `Codable`, string interpolation, collection operations (`map`, `filter`, `reduce`).

## Unsupported Features

Key paths as values, complex macros, C interop (use SkipFFI), `Mirror`/reflection, variadic generics, `UIViewRepresentable`.

## Kotlin Interop

```swift
#if SKIP
import android.content.Context
let context = ProcessInfo.processInfo.androidContext
let uuid = java.util.UUID.randomUUID().toString()
#endif
```

## Diagnosing Failures

```bash
swift test --filter XCSkipTests    # Run transpilation + Kotlin compilation
swift test --filter XCSkipTests -v # Verbose Gradle errors
```

Check `.kt` file paths and line numbers in Gradle error output; cross-reference with Swift source.

## References

- ./references/supported-features.md -- Complete feature support list
