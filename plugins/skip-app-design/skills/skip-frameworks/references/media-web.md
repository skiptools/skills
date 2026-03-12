# Media and Web Frameworks

## SkipWeb — WebView and In-App Browser

### Setup

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-web.git", from: "1.0.0"),
.product(name: "SkipWeb", package: "skip-web"),
```

```yaml
# skip.yml
skip:
  mode: 'transpiled'
  bridging: true

build:
  contents:
    - block: 'dependencies'
      export: false
      contents:
        - 'implementation("androidx.browser:browser:1.9.0")'
        - 'implementation("androidx.webkit:webkit:1.15.0")'
```

### Embedded WebView

```swift
import SkipWeb

struct BrowserView: View {
    @State private var navigator = WebViewNavigator()

    var body: some View {
        WebView(url: URL(string: "https://skip.dev")!, navigator: navigator)
            .toolbar {
                ToolbarItem(placement: .bottomBar) {
                    HStack {
                        Button("Back") { navigator.goBack() }
                        Button("Forward") { navigator.goForward() }
                        Button("Reload") { navigator.reload() }
                    }
                }
            }
    }
}
```

### JavaScript Execution

```swift
navigator.evaluateJavaScript("document.title") { result in
    print("Page title: \(result)")
}
```

### In-App Browser

```swift
// Opens SFSafariViewController (iOS) / Chrome Custom Tabs (Android)
.openWebBrowser(isPresented: $showBrowser, url: URL(string: "https://example.com")!)
```

## SkipAV — Audio and Video

### Setup

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-av.git", from: "1.0.0"),
.product(name: "SkipAV", package: "skip-av"),
```

The `skip.yml` for SkipAV requires several Media3 dependencies — check the [skip-av repository](https://github.com/skiptools/skip-av) for the current configuration.

### Video Playback

```swift
import AVFoundation
import AVKit

struct VideoView: View {
    @State private var player = AVPlayer(url: URL(string: "https://example.com/video.mp4")!)

    var body: some View {
        VideoPlayer(player: player)
            .onAppear { player.play() }
            .onDisappear { player.pause() }
    }
}
```

### Audio Playback

```swift
import AVFoundation

let audioPlayer = try AVAudioPlayer(contentsOf: audioURL)
audioPlayer.play()
audioPlayer.pause()
audioPlayer.stop()
```

### Audio Recording

```swift
import AVFoundation

let recorder = try AVAudioRecorder(url: recordingURL, settings: [:])
recorder.record()
recorder.stop()
```

### Support Levels

- `AVAudioPlayer`: High
- `AVAudioRecorder`: High
- `AVPlayer` / `VideoPlayer`: Medium
- Notifications: `.didPlayToEndTimeNotification`, `.timeJumpedNotification`

### Permissions

Android requires in `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

## SkipMotion — Lottie Animations

### Setup

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-motion.git", "0.0.0"..<"2.0.0"),
.product(name: "SkipMotion", package: "skip-motion"),
```

```yaml
# skip.yml
skip:
  mode: 'transpiled'
  bridging: true

settings:
  contents:
    - block: 'dependencyResolutionManagement'
      contents:
        - block: 'versionCatalogs'
          contents:
            - block: 'create("libs")'
              contents:
                - 'version("lottie-compose", "6.7.1")'
                - 'library("lottie-compose", "com.airbnb.android", "lottie-compose").versionRef("lottie-compose")'

build:
  contents:
    - block: 'dependencies'
      export: false
      contents:
        - 'implementation(libs.lottie.compose)'
```

### Usage

```swift
import SkipMotion

if let url = Bundle.module.url(forResource: "loading", withExtension: "json"),
   let data = try? Data(contentsOf: url) {
    MotionView(lottie: data)
}
```

### Notes

- Renders After Effects animations exported as Lottie JSON
- iOS uses `lottie-ios`; Android uses `lottie-android`
- Early stage: no playback controls (pause/resume/seek) yet
- Adds ~1MB to app size

## SkipDevice — Location, Sensors, Network

### Setup

```swift
// Package.swift
.package(url: "https://source.skip.dev/skip-device.git", "0.0.0"..<"2.0.0"),
.product(name: "SkipDevice", package: "skip-device"),
```

```yaml
# skip.yml
skip:
  bridging: true
```

### Location

```swift
import SkipDevice

// Single location
let location = try await LocationProvider.shared.currentLocation()

// Streaming updates
for try await location in LocationProvider.shared.locationUpdates() {
    print("Lat: \(location.latitude), Lon: \(location.longitude)")
}
```

### Motion Sensors

```swift
for try await data in AccelerometerProvider.shared.accelerometerUpdates() {
    print("X: \(data.x), Y: \(data.y), Z: \(data.z)")
}

for try await data in GyroscopeProvider.shared.gyroscopeUpdates() { ... }
for try await data in MagnetometerProvider.shared.magnetometerUpdates() { ... }
for try await data in BarometerProvider.shared.barometerUpdates() { ... }
```

### Network Reachability

```swift
let isConnected = NetworkReachability.shared.isNetworkReachable
```

### Permissions

**iOS** (xcconfig):
- `NSLocationWhenInUseUsageDescription`
- `NSMotionUsageDescription`

**Android** (AndroidManifest.xml):
- `ACCESS_FINE_LOCATION` or `ACCESS_COARSE_LOCATION`
- `ACCESS_NETWORK_STATE`
- `android.hardware.sensor.barometer` (feature, for barometer)
