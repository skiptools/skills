---
name: skip-testing
description: Writing and running tests in Skip projects. Covers XCTest, Swift Testing, parity testing across iOS and Android, Robolectric, instrumented tests on emulator/device, test harness configuration, and diagnosing cross-platform test failures.
---

# Skip Testing

Skip supports parity testing — running the same tests on both iOS (Swift) and Android (Kotlin/JUnit) from a single test suite.

## How It Works

**Skip Lite**: Swift test files are transpiled to Kotlin JUnit and run via Gradle. A test harness (`XCSkipTests.swift`) is auto-generated if not present.

**Skip Fuse**: Tests are cross-compiled as native Swift and executed on device/emulator via `skip android test`.

## Writing Tests

### XCTest

```swift
import XCTest

final class MyTests: XCTestCase {
    func testLogic() throws {
        XCTAssertEqual(1 + 1, 2)
        XCTAssertTrue(someCondition)
        XCTAssertNotNil(optionalValue)
    }

    func testAsync() async throws {
        let result = try await fetchData()
        XCTAssertFalse(result.isEmpty)
    }
}
```

### Swift Testing (Skip Lite)

```swift
import Testing

@Suite struct MathTests {
    @Test func addition() { #expect(1 + 1 == 2) }
    @Test func inequality() { #expect(3 != 5) }
    @Test func comparisons() {
        #expect(10 > 5)
        #expect(3 < 8)
        #expect(5 >= 5)
    }
    @Test func unwrap() throws {
        let v: Int? = 42
        let _ = try #require(v)
    }
}

// Freestanding @Test functions also supported
@Test func standalone() { #expect(true) }
```

**Supported**: `@Test`, `@Suite`, `#expect` (==, !=, >, <, >=, <=, bool), `#require`.
**Not supported**: Parameterized tests, traits, tags.

## Running Tests

```bash
swift test                                      # All tests
swift test --filter MyModuleTests.MyTests       # Specific class
swift test --filter MyModuleTests/testLogic     # Specific method
swift test --filter XCSkipTests                 # Kotlin only
ANDROID_SERIAL=emulator-5554 swift test         # On emulator
skip android test                               # Fuse CLI mode
skip android test --apk                         # Fuse APK mode
```

## Test Environments

| Environment | Command | Speed | Android APIs |
|------------|---------|-------|-------------|
| Robolectric | `swift test` | Fastest | Simulated |
| Emulator | `ANDROID_SERIAL=… swift test` | Slowest | Full |
| Fuse CLI | `skip android test` | Fast | None |
| Fuse APK | `skip android test --apk` | Moderate | Full |

Note: `#if os(Android)` is `false` under Robolectric. Use `#if os(Android) || ROBOLECTRIC` for both.

## Platform-Specific Tests

```swift
func testIOSOnly() throws {
    #if SKIP
    throw XCTSkip("Not supported on Android")
    #endif
    // iOS-only test
}
```

## Custom Test Harness

Create `XCSkipTests.swift` to override the auto-generated harness:

```swift
#if os(macOS)
import SkipTest

@available(macOS 13, *)
final class XCSkipTests: XCTestCase, XCGradleHarness {
    public func testSkipModule() async throws {
        try await runGradleTests()
    }
}
#endif
```

## Failure Behavior

- **Swift XCTest**: Assertion failure noted, test continues
- **Kotlin JUnit**: Assertion throws exception, test stops

This can cause different failure counts per platform.

## Diagnosing Failures

Kotlin compilation errors appear in `testSkipModule()` output with `.kt` file paths and line numbers. Cross-reference with Swift source.
