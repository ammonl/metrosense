## Project Settings

- **Ticket Provider**: GitHub Issues
- **Branch Format**: `<type>/<ticket-number>` (e.g., `feature/123`)
- **Main Branch**: `main`

## Project Overview

MetroSense is a native iOS app (Swift 5.9, SwiftUI, iOS 17+) that detects when a user is traveling on the Copenhagen Metro using CoreLocation. No third-party dependencies — uses only SwiftUI, CoreLocation, and Combine.

## Development Team

Apple Development Team ID: `E5787F652X`

## Build Commands

```bash
# Build (Debug)
xcodebuild -scheme MetroSense -configuration Debug build

# Build (Release)
xcodebuild -scheme MetroSense -configuration Release build

# Run tests (no test target exists yet)
xcodebuild -scheme MetroSense test

# Open in Xcode
open MetroSense.xcodeproj
```

The project uses XcodeGen (`project.yml` generates `MetroSense.xcodeproj`). No linter or formatter is configured.

## Architecture

**MVVM with Combine** — reactive data flows from CoreLocation through to SwiftUI views:

```
CLLocationManager → LocationService (@Published) → MetroViewModel (Combine) → ContentView (@StateObject)
```

- **`MetroSenseApp.swift`** — App entry point, launches `SplashScreen` as the root view
- **`Views/SplashScreen.swift`** — Animated splash screen that displays the app icon and name, then transitions to `ContentView` after 1.5 seconds
- **`Services/LocationService.swift`** — CLLocationManager wrapper; publishes location, speed, auth status. Filters by accuracy (≤50m) and staleness (≤10s)
- **`ViewModels/MetroViewModel.swift`** — `@MainActor` trip detection logic. Combines location+speed data to drive a state machine: idle → atStation → onMetro → arrived. Selects likely metro line from departure station and direction
- **`ContentView.swift`** — Three-card UI (status, speed, nearest station) styled by trip state
- **`Models/`** — `MetroLine` (M1–M4 with hardcoded stations), `MetroStation` (coordinates + 150m proximity radius), `MetroTripState` (FSM enum with associated data)

## Key Detection Parameters

| Parameter                   | Value                     | Location                |
| --------------------------- | ------------------------- | ----------------------- |
| Metro speed range           | 8.0–25.0 m/s (29–90 km/h) | `MetroLine.swift`       |
| Station proximity           | 150 meters                | `MetroStation.swift`    |
| Location accuracy threshold | ≤200 meters               | `LocationService.swift` |
| Location staleness limit    | 30 seconds                | `LocationService.swift` |
| Distance filter             | 10 meters                 | `LocationService.swift` |

## Permissions

The app requires location access (foreground + background). Configured in `Info.plist` with `UIBackgroundModes: location`.
