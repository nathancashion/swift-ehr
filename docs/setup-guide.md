# Setup Guide

Instructions for setting up the SwiftEHR development environment.

## Prerequisites

- **macOS Ventura 13.5** or later
- **Xcode 15.0** or later ([Mac App Store](https://apps.apple.com/app/xcode/id497799835))
- **Git** (included with Xcode Command Line Tools)

Optional:
- **SwiftLint** — `brew install swiftlint`
- **SwiftFormat** — `brew install swiftformat`
- Physical iOS device for biometric testing (Face ID / Touch ID)

## 1. Clone the Repository

```bash
git clone https://github.com/nathancashion/swift-ehr.git
cd swift-ehr
```

## 2. Open in Xcode

```bash
open SwiftEHR.xcodeproj
```

Xcode will index the project and resolve any dependencies automatically.

## 3. Select a Build Target

1. Click the scheme selector in the toolbar.
2. Choose **SwiftEHR** as the active scheme.
3. Select a run destination:
   - **iPhone 15 Pro** simulator (recommended for development)
   - **iPad Pro 13-inch** simulator (for iPad layout testing)
   - A connected physical device (required for HealthKit/biometrics)
4. Press **⌘R** to build and run.

## 4. Running Tests

### Unit & Integration Tests

```
⌘U  (or Product → Test)
```

Tests use an **in-memory** SwiftData `ModelContainer` so they don't affect on-device data:

```swift
let config = ModelConfiguration(isStoredInMemoryOnly: true)
let container = try ModelContainer(for: schema, configurations: [config])
```

### UI Tests

1. Select the **SwiftEHRUITests** scheme.
2. Choose an iOS Simulator.
3. Press **⌘U**.

UI tests cover critical flows: patient creation, encounter SOAP entry, assessment recording.

## 5. Code Quality Tools

### SwiftLint

SwiftLint runs as an Xcode build phase. To run manually:

```bash
# Install
brew install swiftlint

# Run from project root
swiftlint

# Auto-fix where possible
swiftlint --fix
```

### SwiftFormat

```bash
# Install
brew install swiftformat

# Format all source files
swiftformat SwiftEHR/ SwiftEHRTests/
```

### Adding SwiftLint as a Build Phase

If not already configured:

1. Select the project in the navigator.
2. Select the **SwiftEHR** target.
3. Go to **Build Phases**.
4. Click **+** → **New Run Script Phase**.
5. Add:
   ```bash
   if which swiftlint > /dev/null; then
     swiftlint
   else
     echo "warning: SwiftLint not installed"
   fi
   ```

## 6. Issue Tracking (beads)

This project uses `bd` for issue tracking. To set up:

```bash
# Install beads (if not already installed)
# See https://github.com/beads-dev/beads

# Onboard to the project
bd onboard

# Check for available work
bd ready
```

See `AGENTS.md` for the full workflow.

## 7. Troubleshooting

### Build errors after Xcode update

```
# Clean build folder
⌘⇧K  (or Product → Clean Build Folder)

# Delete derived data
rm -rf ~/Library/Developer/Xcode/DerivedData/SwiftEHR-*
```

### SwiftData migration errors

If the schema has changed and you see migration errors in the simulator:

1. **Reset simulator**: Device → Erase All Content and Settings
2. Or delete the app from the simulator and rebuild

For production, use versioned `Schema` with explicit migration plans.

### Biometrics not working in Simulator

Face ID and Touch ID require special simulator setup:

1. **Features → Face ID → Enrolled** (in Simulator menu)
2. Trigger authentication: **Features → Face ID → Matching Face**

Note: Real biometric hardware testing requires a physical device.

### SwiftLint not found

```bash
# Verify installation
which swiftlint

# If missing
brew install swiftlint

# If Homebrew is not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Git hooks from beads

If you see errors related to git hooks after cloning:

```bash
# Re-initialize beads hooks
bd onboard
```

The `.beads/hooks/` directory contains git hooks that auto-sync issue tracking data.
