---
name: swift-localization
description: Best practices for internationalizing Swift/SwiftUI applications using LocalizedStringResource, String Catalogs (.xcstrings), and type-safe localization patterns. Use when implementing multi-language support, adding new UI strings, or refactoring hardcoded text in Swift apps.
license: MIT
---

# Swift Localization Best Practices

Comprehensive guide for implementing localization in Swift and SwiftUI applications using modern Apple frameworks and type-safe patterns.

## Core Principles

### 1. Never Hardcode User-Facing Strings
All text visible to users must be localized, including:
- UI labels, buttons, and navigation titles
- Error messages and alerts
- Placeholder text and hints
- Accessibility labels and hints
- Status messages and notifications

**Bad:**
```swift
Text("Add Weight")
Button("Save") { ... }
.alert("Error", message: "Something went wrong")
```

**Good:**
```swift
Text(L10n.Common.addWeight)
Button(L10n.Common.saveButton) { ... }
.alert(L10n.Common.errorTitle, message: L10n.Errors.genericMessage)
```

### 2. Use Type-Safe Localization Keys
Create a centralized enum structure (commonly named `L10n`) using `LocalizedStringResource` for compile-time safety.

## Implementation Pattern

### Structure: L10n.swift

Create a hierarchical enum structure organized by feature or screen:

```swift
import Foundation

enum L10n {
    enum Common {
        static let saveButton = LocalizedStringResource(
            "common.button.save",
            defaultValue: "Save"
        )
        static let cancelButton = LocalizedStringResource(
            "common.button.cancel",
            defaultValue: "Cancel"
        )
        static let errorTitle = LocalizedStringResource(
            "common.alert.errorTitle",
            defaultValue: "Error"
        )
    }
    
    enum Dashboard {
        static let navigationTitle = LocalizedStringResource(
            "dashboard.navigation.title",
            defaultValue: "Dashboard"
        )
        static func greeting(_ name: String) -> LocalizedStringResource {
            LocalizedStringResource(
                "dashboard.greeting",
                defaultValue: "Hello, \(name)!"
            )
        }
    }
    
    enum Settings {
        static let navigationTitle = LocalizedStringResource(
            "settings.navigation.title",
            defaultValue: "Settings"
        )
        static let themeTitle = LocalizedStringResource(
            "settings.personalization.theme.title",
            defaultValue: "Theme"
        )
    }
}
```

### Key Naming Convention

Use dot-separated namespaces that mirror your code structure:
- `{feature}.{component}.{element}.{property}`
- Examples:
  - `dashboard.currentWeight` - Simple value
  - `common.button.save` - Reusable button
  - `settings.section.personalization.title` - Nested section title
  - `addEntry.error.invalidWeight` - Error message

### Parameterized Strings

For strings with dynamic content, use functions that return `LocalizedStringResource`:

```swift
enum L10n {
    enum Dashboard {
        static func latestEntry(_ time: String) -> LocalizedStringResource {
            LocalizedStringResource(
                "dashboard.latestEntry",
                defaultValue: "Latest: \(time)"
            )
        }
        
        static func averageEntries(_ count: Int) -> LocalizedStringResource {
            LocalizedStringResource(
                "dashboard.averageEntries",
                defaultValue: "Average of \(count) entries"
            )
        }
    }
}
```

### Usage in SwiftUI Views

Convert `LocalizedStringResource` to `String` using `String(localized:)`:

```swift
struct DashboardView: View {
    var body: some View {
        NavigationStack {
            VStack {
                Text(L10n.Dashboard.navigationTitle)
                Button(L10n.Common.saveButton) {
                    save()
                }
            }
            .navigationTitle(String(localized: L10n.Dashboard.navigationTitle))
        }
    }
}
```

**For string interpolation or concatenation:**
```swift
TextField(
    String(localized: L10n.AddEntry.weightPlaceholder),
    text: $weightText
)
.accessibilityLabel(String(localized: L10n.Accessibility.weightValue))
.accessibilityHint(String(localized: L10n.Accessibility.weightValueHint(unitSymbol)))
```

### Accessibility Localization

Always localize accessibility strings separately:

```swift
enum L10n {
    enum Accessibility {
        static let addWeightEntry = LocalizedStringResource(
            "accessibility.label.addWeightEntry",
            defaultValue: "Add weight entry"
        )
        static let addWeightEntryHint = LocalizedStringResource(
            "accessibility.hint.addWeightEntry",
            defaultValue: "Opens form to log a new weight"
        )
        static func weightValueHint(_ unitSymbol: String) -> LocalizedStringResource {
            LocalizedStringResource(
                "accessibility.hint.weightValue",
                defaultValue: "Enter your current weight in \(unitSymbol)"
            )
        }
    }
}
```

Usage:
```swift
Button {
    showAddEntry = true
} label: {
    Image(systemName: "plus")
}
.accessibilityLabel(String(localized: L10n.Accessibility.addWeightEntry))
.accessibilityHint(String(localized: L10n.Accessibility.addWeightEntryHint))
```

## String Catalogs (.xcstrings)

### File Structure

Modern Swift projects use `.xcstrings` files (String Catalogs) instead of `.strings` files. Xcode automatically generates entries when you use `LocalizedStringResource`.

Location: `{ProjectName}/Localization/Localizable.xcstrings`

Example structure:
```json
{
  "sourceLanguage" : "en",
  "strings" : {
    "common.button.save" : {
      "extractionState" : "manual",
      "localizations" : {
        "en" : {
          "stringUnit" : {
            "state" : "translated",
            "value" : "Save"
          }
        },
        "es" : {
          "stringUnit" : {
            "state" : "translated",
            "value" : "Guardar"
          }
        },
        "fr" : {
          "stringUnit" : {
            "state" : "translated",
            "value" : "Enregistrer"
          }
        }
      }
    }
  },
  "version" : "1.0"
}
```

### Managing String Catalogs in Xcode

1. **Build the project** after adding new `LocalizedStringResource` entries - Xcode will detect them
2. Open `Localizable.xcstrings` in Xcode
3. Use the **String Catalog editor** to add translations
4. Xcode shows:
   - ✓ Translated strings
   - ⚠️ Untranslated strings (needs attention)
   - States: `new`, `translated`, `needs_review`

### Adding New Languages

1. In Xcode: Project Settings → Info → Localizations → `+`
2. Select language (e.g., Spanish, French, German)
3. Xcode adds the language to all `.xcstrings` files
4. Translate strings in the String Catalog editor

## Common Patterns

### Conditional Text
```swift
enum L10n {
    enum Settings {
        static let iCloudSyncEnabled = LocalizedStringResource(
            "settings.data.iCloudSync.enabled",
            defaultValue: "On"
        )
        static let iCloudSyncDisabled = LocalizedStringResource(
            "settings.data.iCloudSync.disabled",
            defaultValue: "Off"
        )
    }
}

// Usage
Text(isEnabled ? L10n.Settings.iCloudSyncEnabled : L10n.Settings.iCloudSyncDisabled)
```

### Error Messages
```swift
enum L10n {
    enum AddEntry {
        static let errorInvalidWeight = LocalizedStringResource(
            "addEntry.error.invalidWeight",
            defaultValue: "Please enter a valid weight"
        )
        static func errorSaveFailure(_ message: String) -> LocalizedStringResource {
            LocalizedStringResource(
                "addEntry.error.saveFailure",
                defaultValue: "Failed to save entry: \(message)"
            )
        }
    }
}

// Usage
.alert(L10n.Common.errorTitle, isPresented: $showingError) {
    Button(L10n.Common.okButton) { showingError = false }
} message: {
    Text(L10n.AddEntry.errorInvalidWeight)
}
```

### Pluralization
For count-dependent strings, use string interpolation:
```swift
static func days(_ count: Int) -> LocalizedStringResource {
    LocalizedStringResource(
        "common.value.days",
        defaultValue: "\(count) days"
    )
}
```

In `.xcstrings`, you can add plural rules:
```json
"common.value.days" : {
  "localizations" : {
    "en" : {
      "variations" : {
        "plural" : {
          "one" : {
            "stringUnit" : {
              "state" : "translated",
              "value" : "%lld day"
            }
          },
          "other" : {
            "stringUnit" : {
              "state" : "translated",
              "value" : "%lld days"
            }
          }
        }
      }
    }
  }
}
```

### Date Formatting
Let Foundation handle date localization:
```swift
// Good - respects user's locale
Text(date, style: .date)
Text(date, format: .dateTime.day().month().year())

// Avoid hardcoded formats
Text(dateFormatter.string(from: date)) // ❌ if format is hardcoded
```

## Migration Strategy

### Converting Hardcoded Strings

1. **Audit for hardcoded strings:**
   ```bash
   # Find potential hardcoded user-facing strings
   grep -r 'Text("' --include="*.swift" .
   grep -r 'Button("' --include="*.swift" .
   grep -r '.alert("' --include="*.swift" .
   ```

2. **Create L10n entries:**
   - Add to appropriate enum section
   - Use descriptive key name
   - Provide clear default value

3. **Replace in views:**
   ```swift
   // Before
   Text("Current Weight")
   
   // After
   Text(L10n.Dashboard.currentWeight)
   ```

4. **Build and verify:**
   - Build project to generate `.xcstrings` entries
   - Open String Catalog to verify keys appear
   - Add translations for all supported languages

### Handling Legacy NSLocalizedString

If migrating from `NSLocalizedString`:
```swift
// Old pattern
NSLocalizedString("key", comment: "Description")

// New pattern
LocalizedStringResource("key", defaultValue: "Default Value")
```

Both work with `.xcstrings`, but `LocalizedStringResource` is preferred for SwiftUI.

## Testing Localization

### Pseudo-localization
Test string length and layout issues:
1. Xcode → Product → Scheme → Edit Scheme
2. Run → Options → App Language → "Double-Length Pseudo-language"
3. Strings appear doubled to simulate longer translations

### Language Testing
1. Simulator: Settings → Language & Region → Preferred Languages
2. Xcode scheme: Edit Scheme → Run → App Language → Select language
3. Verify:
   - All strings are translated
   - No truncation or layout issues
   - Right-to-left (RTL) languages display correctly

### Automated Checks
```swift
// Unit test to ensure no hardcoded strings in views
func testNoHardcodedStrings() {
    let source = try! String(contentsOfFile: "DashboardView.swift")
    let pattern = #"Text\("[^L]"#
    let regex = try! NSRegularExpression(pattern: pattern)
    let matches = regex.matches(in: source, range: NSRange(source.startIndex..., in: source))
    XCTAssertEqual(matches.count, 0, "Found hardcoded Text strings")
}
```

## Best Practices Summary

✅ **Do:**
- Organize L10n enum by feature/screen
- Use `LocalizedStringResource` for type safety
- Provide descriptive default values
- Localize accessibility strings separately
- Use functions for parameterized strings
- Let Foundation handle date/number formatting
- Test with pseudo-localization and RTL languages

❌ **Don't:**
- Hardcode user-facing strings directly in views
- Use string concatenation for sentences (breaks translation)
- Assume English word order works in all languages
- Skip accessibility localization
- Use generic key names like "title1", "label2"
- Forget to build after adding new keys

## Tooling

### Xcode String Catalog Editor
- **Filter:** Search/filter strings by state (new, translated, needs review)
- **Bulk edit:** Select multiple strings to mark as reviewed
- **Export/Import:** File → Export/Import Localizations for external translation

### Command Line
```bash
# Extract strings from code (legacy .strings format)
genstrings -o en.lproj *.swift

# Export for translation
xcodebuild -exportLocalizations -project MyApp.xcodeproj -localizationPath ./Localizations

# Import translations
xcodebuild -importLocalizations -project MyApp.xcodeproj -localizationPath ./Localizations/fr.xcloc
```

## Resources

- [Apple Localization Documentation](https://developer.apple.com/documentation/xcode/localization)
- [LocalizedStringResource API](https://developer.apple.com/documentation/foundation/localizedstringresource)
- [String Catalogs](https://developer.apple.com/documentation/xcode/localizing-and-varying-text-with-a-string-catalog)
- [WWDC: Discover String Catalogs](https://developer.apple.com/videos/play/wwdc2023/10155/)

## Quick Reference

| Task | Pattern |
|------|---------|
| Simple string | `static let title = LocalizedStringResource("key", defaultValue: "Title")` |
| Parameterized | `static func greeting(_ name: String) -> LocalizedStringResource { ... }` |
| In SwiftUI | `Text(L10n.Feature.label)` |
| With String conversion | `String(localized: L10n.Feature.label)` |
| Navigation title | `.navigationTitle(String(localized: L10n.Feature.title))` |
| Accessibility | `.accessibilityLabel(String(localized: L10n.Accessibility.label))` |
| Alert title | `.alert(L10n.Common.errorTitle, message: ...)` |

---

*This skill provides general best practices for Swift localization. Adapt patterns to your project's specific architecture and requirements.*
