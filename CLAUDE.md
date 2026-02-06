# CLAUDE.md — Claude Code Instructions

Instructions for Claude Code, Cursor, and other Claude-powered development tools.

## Quick Context

- **iOS app** — SwiftUI, iOS 17+, Swift 5.9+
- **Architecture** — MVVM (Model-View-ViewModel)
- **Offline-first** — Local storage primary, Firebase optional
- **Xcode project** — Open `ios/` folder in Xcode

## Before Writing Code

1. **Read `ARCHITECTURE.md`** — Contains patterns and conventions
2. **Check `AppConfiguration.swift`** — Feature flags control behavior
3. **Reference `Library/`** — Canonical example for new features

## Code Style

```swift
// MARK: - Properties
@Published var items: [Item] = []
@Published var isLoading = false

// MARK: - Initialization
init() { }

// MARK: - Public Methods
func fetch() { }

// MARK: - Private Methods
private func saveToLocal() { }
```

## SwiftUI Patterns

### ViewModels — Use ObservableObject
```swift
// ✅ Correct
class MyViewModel: ObservableObject {
    @Published var data: String = ""
}

// ❌ Wrong — @Observable is iOS 17 macro, we use ObservableObject
@Observable class MyViewModel { }
```

### Views — Use @StateObject for owned ViewModels
```swift
struct MyView: View {
    @StateObject private var viewModel = MyViewModel.shared
    
    var body: some View {
        Text(viewModel.data)
    }
    .task {
        Analytics.trackScreenView("MyView")
    }
}
```

### Firebase — Always guard with both checks
```swift
#if canImport(Firebase)
import Firebase
#endif

func save() {
    saveLocally()  // Always local first
    
    #if canImport(Firebase)
    if AppConfiguration.useFirebase {
        saveToCloud()
    }
    #endif
}
```

## File Organization

New features go in `ios/Sources/YourFeature/`:
```
YourFeature/
├── Models/
│   └── YourModel.swift
├── ViewModels/
│   └── YourViewModel.swift
└── Views/
    ├── YourView.swift
    └── YourDetailView.swift
```

## Common Operations

### Create a new View
```swift
import SwiftUI

struct NewView: View {
    var body: some View {
        VStack {
            // Content
        }
    }
    .task {
        Analytics.trackScreenView("NewView")
    }
}

#Preview {
    NewView()
}
```

### Create a new ViewModel
```swift
import Foundation

class NewViewModel: ObservableObject {
    @Published var items: [Item] = []
    @Published var isLoading = false
    
    static let shared = NewViewModel()
    
    private init() {
        loadFromLocal()
    }
    
    private func loadFromLocal() {
        // Load from UserDefaults
    }
}
```

### Add UserDefaults storage
```swift
private let storageKey = "my_items"

private func saveToLocal() {
    guard let data = try? JSONEncoder().encode(items) else { return }
    UserDefaults.standard.set(data, forKey: storageKey)
}

private func loadFromLocal() {
    guard let data = UserDefaults.standard.data(forKey: storageKey),
          let decoded = try? JSONDecoder().decode([Item].self, from: data) else {
        return
    }
    items = decoded
}
```

## Testing

```bash
# Verify Swift compilation (no Xcode needed)
cd ios && swift build

# Full test requires Xcode
open ios/MyApp.xcodeproj  # or use Xcode
```

## Key Files Reference

| What | Where |
|------|-------|
| Feature flags | `ios/Sources/App/AppConfiguration.swift` |
| App entry | `ios/Sources/App/MyApp.swift` |
| Tab bar | `ios/Sources/TabBar/MainTabView.swift` |
| Theme colors | `ios/Sources/UI/Theme/AppColors.swift` |
| Auth logic | `ios/Sources/Auth/AuthManager.swift` |
| Settings | `ios/Sources/User/SettingsViewModel.swift` |
| Analytics | `ios/Sources/Analytics/Analytics.swift` |

## Conventions

- **Models**: Singular (`LibraryEntry` not `LibraryEntries`)
- **ViewModels**: `FeatureViewModel`
- **Views**: `FeatureView`, `FeatureDetailView`, `FeatureRowView`
- **Managers**: Singletons with `.shared`
- **MARK comments**: Group code sections

## Don't

- Use `@Observable` macro (we use `ObservableObject`)
- Skip Analytics tracking on views
- Access storage directly from Views
- Ignore `AppConfiguration` flags
- Add Firebase code without `#if canImport()` guards
- Refactor unrelated code

## Dependencies

Managed via Swift Package Manager in `ios/Package.swift`:
- RevenueCat (subscriptions)
- TelemetryDeck (analytics)
- Firebase (optional)
- MarkdownUI (content rendering)
- ConfettiSwiftUI, SwiftUI-Shimmer (effects)

Don't add new dependencies without updating Package.swift.
