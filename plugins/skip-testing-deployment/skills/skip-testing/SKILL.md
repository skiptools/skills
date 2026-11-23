---
name: skip-testing
description: Writing and running tests in Skip projects. Covers XCTest, Swift Testing, the auto-generated `XCSkipTests` parity harness, the three test runners (`swift test`, `skip android test` CLI mode, `skip android test --apk` mode), the `skip test` parity-report command that produces side-by-side iOS vs. Android results, Robolectric vs. instrumented testing on emulator/device, custom test harness configuration, and diagnosing cross-platform test failures. Use when writing tests, deciding which runner to use, or debugging a Kotlin compile failure in `XCSkipTests.testSkipModule`.
---

# Skip Testing

Skip supports parity testing — running the same tests on both iOS (Swift) and Android (Kotlin/JUnit) from a single test suite — through several different commands depending on the mode and how thorough you need to be.

## How It Works

**Skip Lite**: Swift test files are transpiled to Kotlin/JUnit and run via Gradle. A test harness (`XCSkipTests.swift`) is auto-generated if not present and runs the Kotlin side via Robolectric by default.

**Skip Fuse**: Swift test files are compiled natively for Android via the Swift Android SDK and executed on device/emulator with `skip android test`. The auto-generated `XCSkipTests` harness does not apply to Fuse; use `skip android test` directly.

## The four test runners

| Runner | Command | Where the Android side runs | Android APIs | Resource bundles | Speed |
|---|---|---|---|---|---|
| **Lite Robolectric** | `swift test` | JVM on the host via Gradle + `XCSkipTests` harness | Simulated by Robolectric | Yes | Fastest |
| **Lite Instrumented** | `ANDROID_SERIAL=… swift test` | Android emulator / device via Gradle | Full | Yes | Slowest |
| **Fuse CLI** | `skip android test` | Android emulator / device via `adb shell` | None — bare Linux exec, no JVM/JNI | Yes (sidecar `.resources`) | Fast |
| **Fuse APK** | `skip android test --apk` | Android emulator / device, packaged as an APK | Full (real app process with JNI) | **No** — `Bundle.module` doesn't work | Moderate |

Two important caveats:

- **APK mode is Swift Testing-only.** XCTest tests are skipped under `--apk`; use `@Test` / `@Suite` annotations from the Swift Testing module for tests you want to run in APK mode.
- **Resource bundles don't work in APK mode.** If your tests rely on `Bundle.module.url(forResource:withExtension:)`, they will fail under `--apk`. Use CLI mode (which keeps a sidecar `.resources` directory next to the test binary) or restructure the tests to read fixtures from the file system / a network mock.

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

### Swift Testing (Lite and Fuse)

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

// Freestanding @Test functions are also supported
@Test func standalone() { #expect(true) }
```

**Supported**: `@Test`, `@Suite`, `#expect` (==, !=, >, <, >=, <=, bool), `#require`, and `@Test(arguments:)` with single collections, tuples, or two collections for all combinations, including async tests.

**Not supported**: Traits, tags, generic/variadic/`inout` test parameters, and arguments depending on instance state. For stateful suites, prefer classes to avoid additional struct constructors that JUnit rejects.

## Running Tests

```bash
swift test                                       # Lite: Swift host + transpiled JUnit (Robolectric)
swift test --filter MyModuleTests.MyTests        # one Swift test class
swift test --filter MyModuleTests/testLogic      # one Swift test method
swift test --filter XCSkipTests                  # only the Kotlin-transpiled side
ANDROID_SERIAL=emulator-5554 swift test          # Lite, instrumented on emulator

skip android test                                # Fuse, CLI mode
skip android test --apk                          # Fuse, APK mode
skip android test --android-serial emulator-5554 # scope Fuse test to one device
skip android test --testing-library swift-testing # only @Test / @Suite
skip android test --event-stream-output-path tests.jsonl  # JSON event stream
```

## The `skip test` parity-report command

`skip test` is a higher-level command that builds and runs **both** the Swift and the transpiled Kotlin tests, then produces a side-by-side parity report so you can see which tests pass on which platform at a glance.

```bash
skip test                                # parity report for the current project
skip test --android-serial emulator-5554 # parity report with instrumented Android side
skip test --xunit results.xml            # write XUnit output for CI
skip test --junit junit-reports/         # write a JUnit report directory
skip test -c release                     # release-configuration parity
```

The output is a two-column table with the test name on the left and pass/fail per platform on the right. Use this when you suspect a transpilation regression: a test that passes on Swift but fails on Kotlin (or vice versa) is exactly the parity failure mode `skip test` is designed to surface.

`swift test` is fine for everyday iteration. `skip test` is the right pre-commit / CI gate when you want to be sure both sides agree.

## Test Environments

Note: `#if os(Android)` is `false` under Robolectric. Use `#if os(Android) || ROBOLECTRIC` if you need code to run in both real Android *and* Robolectric.

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

When a Kotlin compile error references a transpilation pattern (integer overflow, Double-vs-Int comparison, `Array.remove(atOffsets:)` outside SwiftUI, `Foundation.sqrt` qualification, etc.), the underlying rule is in [skip-lite-transpilation](../../../skip-app-design/skills/skip-lite-transpilation/SKILL.md) — fix the Swift source per that skill and re-run `swift test --filter XCSkipTests` to confirm.

## UI tests are a separate concern

`swift test` / `skip test` cover business logic and parity tests. They do **not** drive the running app — they don't tap buttons, type into URL bars, or take screenshots of the rendered UI. For that, use the inner loop in the [skip-ui-automation](../skip-ui-automation/SKILL.md) skill:

- `skip app launch` to install and run the app on the iOS Simulator and Android emulator together.
- [Maestro](https://maestro.dev) YAML flows under `.maestro/` to script gestures and assertions against the live UI.
- `.accessibilityIdentifier(...)` on every interactive view so flows survive translation and refactor.

Logic tests catch transpilation bugs and regressions; Maestro flows catch rendering, navigation, and layout regressions. Run both.
