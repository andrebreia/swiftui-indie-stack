# AGENTS.md — AI Collaboration Guide

This file helps AI assistants produce high-quality code for this SwiftUI iOS project.

## Project Overview

**SwiftUI Indie Stack** — A production-ready iOS app template with offline-first architecture. Built for iOS 17+, Swift 5.9+.

Key characteristics:
- **MVVM architecture** (Model-View-ViewModel)
- **Offline-first** — Local storage is source of truth, cloud sync is optional
- **Feature flags** — Everything toggleable via `AppConfiguration.swift`
- **Conditional compilation** — Optional dependencies use `#if canImport()`

## Critical Files

| File | Purpose |
|------|---------|
| `ios/Sources/App/AppConfiguration.swift` | Feature flags, API keys, URLs |
| `ARCHITECTURE.md` | Design patterns and conventions |
| `CUSTOMIZATION.md` | How to customize features |

**Read these before making changes.**

## Project Structure

```
ios/Sources/
├── App/          # Entry point, configuration
├── Auth/         # Authentication (optional Firebase)
├── User/         # Settings, Firestore manager
├── Streak/       # Streak system (local or cloud)
├── Paywall/      # RevenueCat subscriptions
├── Analytics/    # TelemetryDeck wrapper
├── Library/      # GitHub-based CMS ← CANONICAL EXAMPLE
├── TabBar/       # Custom tab bar
├── Onboarding/   # Onboarding flow
├── UI/           # Theme, components, modifiers
└── Utilities/    # Logging, haptics, helpers
```

**Reference `Library/` when creating new features** — it's the most complete example.

## Code Patterns

### Feature Structure
```
YourFeature/
├── Models/       # Codable structs
├── ViewModels/   # ObservableObject + @Published
└── Views/        # SwiftUI views
```

### ViewModel Template
```swift
class YourViewModel: ObservableObject {
    @Published var items: [YourItem] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    static let shared = YourViewModel()
    
    func save(_ item: YourItem) {
        // 1. Local first (instant)
        saveToLocal(item)
        
        // 2. Cloud sync (if enabled)
        #if canImport(Firebase)
        if AppConfiguration.useFirebase {
            Task { try? await saveToFirestore(item) }
        }
        #endif
    }
}
```

### View Template
```swift
struct YourView: View {
    @StateObject private var viewModel = YourViewModel.shared
    
    var body: some View {
        // UI here
    }
    .task {
        Analytics.trackScreenView("YourView")
    }
}
```

## Rules

### Do
- Use `ObservableObject` + `@Published` (not `@Observable`)
- Add `Analytics.trackScreenView()` to all views
- Guard Firebase code with both `#if canImport()` AND runtime check
- Save locally first, sync to cloud second
- Match existing naming conventions
- Keep changes minimal and focused

### Don't
- Use `@Observable` (iOS 17 only, this template uses ObservableObject for compatibility)
- Access storage directly from Views (always go through ViewModel)
- Refactor unrelated code when making changes
- Add dependencies without updating `Package.swift`
- Ignore feature flags in `AppConfiguration`

## Conditional Firebase Pattern

**Always use both compile-time AND runtime checks:**

```swift
#if canImport(Firebase)
import Firebase
#endif

func someMethod() {
    saveLocally()
    
    #if canImport(Firebase)
    if AppConfiguration.useFirebase {
        saveToFirestore()
    }
    #endif
}
```

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Models | Singular noun | `LibraryEntry`, `StreakData` |
| ViewModels | Feature + ViewModel | `LibraryViewModel` |
| Views | Descriptive + View | `LibraryDetailView` |
| Managers | Feature + Manager | `PaywallManager` |
| Providers | Feature + Provider | `StreakDataProvider` |

## Key Singletons

| Singleton | Purpose |
|-----------|---------|
| `AuthManager.shared` | Authentication state |
| `PaywallManager.shared` | Subscription state |
| `SettingsViewModel.shared` | User settings |
| `StreakDataProvider.shared` | Streak display |
| `FirestoreManager.shared` | Firestore ops |

## Testing Changes

```bash
cd ios
swift build  # Verify compilation
```

For full testing, open `ios/` in Xcode and run on simulator.

## Common Tasks

### Add a new feature
1. Create folder structure: `Sources/YourFeature/{Models,ViewModels,Views}/`
2. Follow the `Library/` pattern
3. Add to `MainTabView.swift` if it needs a tab
4. Add Analytics tracking

### Add a new setting
1. Add property to `SettingsViewModel.swift`
2. Add UI in settings view
3. If cloud sync needed, update `FirestoreManager`

### Add a new screen
1. Create View file in appropriate feature folder
2. Add Analytics: `.task { Analytics.trackScreenView("ScreenName") }`
3. Add navigation from parent view

---

When in doubt, read the existing code first. Match the style exactly.
