# Live Activities (iOS 16.1+)

The SDK supports iOS Live Activities for real-time workout tracking on the Lock Screen and Dynamic Island (iPhone 14 Pro and later).

> Back to [iOS Integration Guide](README.md)

## Table of Contents

- [What It Provides](#what-it-provides)
- [Prerequisites](#prerequisites)
- [What's Required vs Optional](#whats-required-vs-optional)
- [Setup Overview](#setup-overview)
- [Step 1: Create Widget Extension Target](#step-1-create-widget-extension-target)
- [Step 2: Main App Info.plist](#step-2-main-app-infoplist)
- [Step 3: Widget Extension Files](#step-3-widget-extension-files)
- [Step 4: Widget Extension Info.plist](#step-4-widget-extension-infoplist)
- [Step 5: Widget Extension Entitlements](#step-5-widget-extension-entitlements)
- [Step 6: Assets.xcassets (Required)](#step-6-assetsxcassets-required)
- [Step 7: Configure Target Membership](#step-7-configure-target-membership)
- [Step 8: Verify Embedding](#step-8-verify-embedding)
- [Customization](#customization)
- [Troubleshooting Live Activities](#troubleshooting-live-activities)
- [Full Implementation Reference](#full-implementation-reference)

---

## What It Provides

When a user starts a workout (Walk, Run, Cycling, or Cardio), the SDK can display a Live Activity showing:

- Workout name and elapsed time
- Heart rate with HR zone bar
- Secondary metric (distance or active points)
- Pause/resume state
- Band disconnect warning

The SDK handles all the real-time data flow automatically. You only need to create the Widget Extension and provide the SwiftUI UI files described below.

## Prerequisites

- iOS deployment target 14.0+ (for your main app)
- Xcode 14.0 or later
- Activity Tracking module enabled
- Valid Apple Developer account

> **Note:** Live Activities require iOS 16.1+ at runtime. Your app can still support older iOS versions — the SDK gracefully skips Live Activities on devices running older iOS.

## What's Required vs Optional

| Component | Required | Notes |
|-----------|:--------:|-------|
| Widget Extension target (`liveworkout`) | Yes | Must be named `liveworkout` with iOS 16.1 minimum deployment |
| `LiveWorkoutAttributes.swift` | Yes | Shared data contract — must compile in **both** app and widget targets |
| `liveworkoutBundle.swift` | Yes | Widget entry point |
| `liveworkout.swift` | Yes | Placeholder static widget required by Xcode |
| `liveworkoutLiveActivity.swift` | Yes | SwiftUI UI for Lock Screen and Dynamic Island |
| `NSSupportsLiveActivities` in main app Info.plist | Yes | Enables Live Activity support |
| `NSSupportsLiveActivitiesFrequentUpdates` in main app Info.plist | Yes | Enables high-frequency data updates |
| Push Notifications capability (both targets) | Yes | Required for activity updates |
| Assets.xcassets in widget extension | Yes | Widget won't compile without it |
| Custom SwiftUI styling | Optional | You can modify colors, fonts, and layout in the widget code (see [Customization](#customization)) |

## Setup Overview

| Step | Action |
|------|--------|
| 1 | Create Widget Extension target named `liveworkout` (min deployment iOS 16.1) |
| 2 | Add `NSSupportsLiveActivities` and `NSSupportsLiveActivitiesFrequentUpdates` to main app Info.plist |
| 3 | Add the Swift files to the widget extension (see [Full Implementation Reference](#full-implementation-reference)) |
| 4 | Set `LiveWorkoutAttributes.swift` target membership to **both** Runner + liveworkout |
| 5 | Add Push Notifications capability to both targets |
| 6 | Add Assets.xcassets to widget extension |
| 7 | Verify widget extension is embedded in your app |
| 8 | Build and test |

## Step 1: Create Widget Extension Target

1. Open your iOS project in Xcode
2. Select your project file in the Navigator (top-level blue icon)
3. Click the **+** button at the bottom of the targets list
4. Select **Widget Extension** > **Next**
5. Configure:
   - Product Name: `liveworkout`
   - Bundle Identifier: `<your.app.bundle.id>.liveworkout`
   - Include Live Activity: **Check**
   - Include Configuration App Intent: **Uncheck**
6. Click **Finish**
7. When prompted "Activate 'liveworkout' scheme?", click **Cancel**
8. Set the widget target's minimum deployment to **iOS 16.1**

Delete all the auto-generated placeholder files that Xcode created in the `liveworkout` folder — we will replace them with the files below.

## Step 2: Main App Info.plist

Add these keys to your **main app's** `Info.plist`:

```xml
<key>NSSupportsLiveActivities</key>
<true/>
<key>NSSupportsLiveActivitiesFrequentUpdates</key>
<true/>
```

## Step 3: Widget Extension Files

Create the following files in your `liveworkout` widget extension. The full source code for each file is in the [Full Implementation Reference](#full-implementation-reference) section below.

| File | Target membership | Purpose |
|------|-------------------|---------|
| `LiveWorkoutAttributes.swift` | Runner **+** liveworkout | Shared data contract between app and widget |
| `liveworkoutBundle.swift` | liveworkout only | Widget bundle entry point (`@main`) |
| `liveworkout.swift` | liveworkout only | Placeholder static widget required by Xcode |
| `liveworkoutLiveActivity.swift` | liveworkout only | Lock Screen and Dynamic Island UI |

**`LiveWorkoutAttributes.swift`** defines the data model the SDK uses to communicate workout state to the widget. It includes:
- `ContentState` — live-updating fields: metrics, heart rate, pause state, band connection status
- Static attributes — activity ID, workout name, SF Symbol, start date

> **Important:** This file must compile in **both** targets. All other files belong to the widget extension only.

## Step 4: Widget Extension Info.plist

The widget extension's `Info.plist` should contain:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSExtension</key>
    <dict>
        <key>NSExtensionPointIdentifier</key>
        <string>com.apple.widgetkit-extension</string>
    </dict>
    <key>NSSupportsLiveActivities</key>
    <true/>
</dict>
</plist>
```

## Step 5: Widget Extension Entitlements

Create `liveworkout.entitlements` with Push Notifications:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>aps-environment</key>
    <string>development</string>
</dict>
</plist>
```

Also add Push Notifications capability to your **main app** target (Signing & Capabilities > + Capability > Push Notifications).

## Step 6: Assets.xcassets (Required)

The widget extension **must** have an `Assets.xcassets` folder or it will fail to compile. Create the folder with at least these files:

```
Assets.xcassets/
├── Contents.json
├── AccentColor.colorset/
│   └── Contents.json
├── AppIcon.appiconset/
│   └── Contents.json
└── WidgetBackground.colorset/
    └── Contents.json
```

Each `Contents.json` should at minimum contain:

```json
{
  "info" : {
    "author" : "xcode",
    "version" : 1
  }
}
```

For `AccentColor.colorset/Contents.json` and `WidgetBackground.colorset/Contents.json`:

```json
{
  "colors" : [
    {
      "idiom" : "universal"
    }
  ],
  "info" : {
    "author" : "xcode",
    "version" : 1
  }
}
```

## Step 7: Configure Target Membership

**Critical:** `LiveWorkoutAttributes.swift` must be compiled into **both** targets:

| File | Your App Target | liveworkout |
|------|:---------------:|:-----------:|
| `LiveWorkoutAttributes.swift` | Yes | Yes |
| `liveworkoutLiveActivity.swift` | No | Yes |
| `liveworkout.swift` | No | Yes |
| `liveworkoutBundle.swift` | No | Yes |

To verify: select the file in Xcode > File Inspector (right sidebar) > check Target Membership.

## Step 8: Verify Embedding

1. Select your main app target > General > Frameworks, Libraries, and Embedded Content
2. Verify `liveworkout.appex` appears
3. In Build Phases, verify "Embed Foundation Extensions" contains `liveworkout.appex`

> **Tip:** If you get a "Cycle inside Runner" build error, drag the "Embed Foundation Extensions" phase **before** the CocoaPods script phases in Build Phases.

## Customization

The Live Activity UI is defined entirely in `liveworkoutLiveActivity.swift`, which you own and can modify. Customization options include:

- **Colors:** The default UI uses a dark gradient background (`#1F1F1F` to black). Change the `LinearGradient` stops in the Lock Screen view and the accent/tint colors in the Dynamic Island.
- **Layout:** Rearrange, resize, or restyle the metric rows, heart rate zone bar, and timer display.
- **Fonts:** Modify font weights, sizes, and styles on any text element.
- **Metrics displayed:** The SDK provides heart rate, a secondary metric (distance or active points), elapsed time, and pause/connection state. You choose how and where to display them.
- **Dynamic Island regions:** Customize what appears in the compact leading/trailing, expanded, and minimal views.

The data contract (`LiveWorkoutAttributes` and `ContentState`) must remain unchanged — the SDK writes to these fields. Everything else in the widget's SwiftUI code is yours to customize.

## Troubleshooting Live Activities

- **Live Activity not appearing:** Check iOS version is 16.1+. Verify `NSSupportsLiveActivities` is in both the widget and main app Info.plist. Check Focus mode isn't hiding Live Activities.
- **Widget shows blank/crashes:** Verify `LiveWorkoutAttributes.swift` is in both targets. Clean build and reinstall.
- **"No provisioning profile" error:** Enable Push Notifications for both App IDs in Apple Developer Portal, then refresh signing profiles in Xcode.
- **"Cycle inside Runner" build error:** Move "Embed Foundation Extensions" build phase before CocoaPods script phases.

---

## Full Implementation Reference

The complete source code for all widget extension files. Copy these into your `liveworkout` target.

### LiveWorkoutAttributes.swift

**Target membership: Runner + liveworkout**

```swift
import Foundation
import ActivityKit

@available(iOS 16.1, *)
struct LiveWorkoutAttributes: ActivityAttributes {
    public struct Metric: Codable, Hashable {
        var label: String
        var value: String
    }

    public struct ContentState: Codable, Hashable {
        var metrics: [Metric]
        var secondaryMetricKind: String?
        var timerStartDate: Date?
        var heartRateBpm: Int?
        var maxHeartRateBpm: Int?
        var isPaused: Bool
        var isBandConnected: Bool
        var disconnectedMessage: String?
        var pausedMessage: String?
        var staleMessage: String?
    }

    var activityId: String
    var name: String
    var sfSymbolName: String?
    var startDate: Date
}
```

### liveworkoutBundle.swift

**Target membership: liveworkout only**

```swift
import WidgetKit
import SwiftUI

@main
struct liveworkoutBundle: WidgetBundle {
    var body: some Widget {
        liveworkout()
        liveworkoutLiveActivity()
    }
}
```

### liveworkout.swift

**Target membership: liveworkout only** — Placeholder static widget required by Xcode. This widget is never shown to users; Xcode requires at least one static widget in any widget bundle.

```swift
import WidgetKit
import SwiftUI

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date(), emoji: "🏃")
    }

    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> ()) {
        completion(SimpleEntry(date: Date(), emoji: "🏃"))
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            entries.append(SimpleEntry(date: entryDate, emoji: "🏃"))
        }
        completion(Timeline(entries: entries, policy: .atEnd))
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
    let emoji: String
}

struct liveworkoutEntryView : View {
    var entry: Provider.Entry
    var body: some View {
        VStack {
            Text("Time:")
            Text(entry.date, style: .time)
        }
    }
}

struct liveworkout: Widget {
    let kind: String = "liveworkout"
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            if #available(iOS 17.0, *) {
                liveworkoutEntryView(entry: entry)
                    .containerBackground(.fill.tertiary, for: .widget)
            } else {
                liveworkoutEntryView(entry: entry)
                    .padding()
                    .background()
            }
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}
```

### liveworkoutLiveActivity.swift

**Target membership: liveworkout only** — Lock Screen and Dynamic Island UI.

```swift
import ActivityKit
import WidgetKit
import SwiftUI

struct liveworkoutLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: LiveWorkoutAttributes.self) { context in
            // Lock screen / banner UI
            ZStack {
                LinearGradient(
                    stops: [
                        .init(color: Color(hex: 0x1F1F1F), location: 0.0),
                        .init(color: Color(hex: 0x171717), location: 0.55),
                        .init(color: Color.black, location: 1.0),
                    ],
                    startPoint: .top,
                    endPoint: .bottom
                )
                .ignoresSafeArea()

                VStack(alignment: .leading, spacing: 10) {
                    HStack(alignment: .firstTextBaseline, spacing: 8) {
                        Text(context.attributes.name)
                            .font(.headline)
                            .foregroundStyle(.white)
                            .lineLimit(1)

                        Spacer(minLength: 8)

                        if context.state.isPaused {
                            Text(context.state.pausedMessage ?? "Paused")
                                .font(.headline)
                                .fontWeight(.bold)
                                .foregroundStyle(.white)
                                .multilineTextAlignment(.trailing)
                                .frame(minWidth: 72, alignment: .trailing)
                        } else {
                            Text((context.state.timerStartDate ?? context.attributes.startDate), style: .timer)
                                .font(.headline)
                                .fontWeight(.bold)
                                .monospacedDigit()
                                .foregroundStyle(.white)
                                .multilineTextAlignment(.trailing)
                                .frame(minWidth: 72, alignment: .trailing)
                        }
                    }

                    if !context.state.isBandConnected {
                        Text(context.state.disconnectedMessage ?? "Band disconnected, tap to open app and reconnect.")
                            .font(.callout)
                            .foregroundStyle(Color.white.opacity(0.78))
                            .lineLimit(3)
                            .multilineTextAlignment(.leading)
                            .frame(maxWidth: .infinity, alignment: .leading)
                    } else {
                        if let maxHr = context.state.maxHeartRateBpm, maxHr > 0 {
                            HrZonesBar(heartRateBpm: context.state.heartRateBpm, maxHeartRateBpm: maxHr)
                                .frame(height: 12)
                                .padding(.vertical, 8)
                        }

                        HrAndApRow(
                            heartRateTitle: context.state.heartRateTitleForRow,
                            heartRateValue: context.state.heartRateDisplayValueForRow,
                            secondaryTitle: context.state.secondaryTitleForRow,
                            secondaryValue: context.state.secondaryDisplayValueForRow,
                            secondaryKind: context.state.secondaryMetricKindForRow
                        )
                    }
                }
                .padding(18)
            }
            .activityBackgroundTint(.clear)
            .activitySystemActionForegroundColor(.white)

        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading) {
                    if context.state.isPaused {
                        Image(systemName: "pause.fill")
                            .font(.caption).fontWeight(.bold).foregroundStyle(.white)
                    } else {
                        Text((context.state.timerStartDate ?? context.attributes.startDate), style: .timer)
                            .font(.caption).fontWeight(.bold).monospacedDigit().foregroundStyle(.white)
                    }
                }
                DynamicIslandExpandedRegion(.trailing) {
                    HStack(spacing: 4) {
                        Image(systemName: "heart.fill").font(.caption2).foregroundStyle(Color.white.opacity(0.78))
                        Text(context.state.heartRateValueWithoutUnit)
                            .font(.caption).fontWeight(.bold).monospacedDigit().foregroundStyle(.white)
                    }
                }
                DynamicIslandExpandedRegion(.bottom) {
                    if !context.state.isBandConnected {
                        Text(context.state.disconnectedMessage ?? "Band disconnected, tap to open app and reconnect.")
                            .font(.caption).foregroundStyle(Color.white.opacity(0.78)).lineLimit(2)
                    } else {
                        VStack(alignment: .leading, spacing: 8) {
                            if let maxHr = context.state.maxHeartRateBpm, maxHr > 0 {
                                HrZonesBar(heartRateBpm: context.state.heartRateBpm, maxHeartRateBpm: maxHr)
                                    .frame(height: 12).padding(.vertical, 6)
                            }
                            HrAndApRow(
                                heartRateTitle: context.state.heartRateTitleForRow,
                                heartRateValue: context.state.heartRateDisplayValueForRow,
                                secondaryTitle: context.state.secondaryTitleForRow,
                                secondaryValue: context.state.secondaryDisplayValueForRow,
                                secondaryKind: context.state.secondaryMetricKindForRow,
                                valueFont: .caption, labelFont: .caption2, dividerHeight: 18
                            )
                        }
                    }
                }
            } compactLeading: {
                if context.state.isPaused {
                    Image(systemName: "pause.fill").font(.caption2).fontWeight(.bold).foregroundStyle(.white)
                } else {
                    Text("00:00").font(.caption2).fontWeight(.bold).monospacedDigit().hidden()
                        .overlay(alignment: .leading) {
                            Text((context.state.timerStartDate ?? context.attributes.startDate), style: .timer)
                                .font(.caption2).fontWeight(.bold).monospacedDigit().foregroundStyle(.white)
                        }
                }
            } compactTrailing: {
                HStack(spacing: 3) {
                    Image(systemName: "heart.fill").font(.caption2).foregroundStyle(Color.white.opacity(0.78))
                    Text(context.state.heartRateValueWithoutUnit)
                        .font(.caption2).fontWeight(.bold).monospacedDigit().foregroundStyle(.white)
                }
            } minimal: {
                if context.state.isPaused {
                    Image(systemName: "pause.fill").font(.caption2).fontWeight(.bold).foregroundStyle(.white)
                } else {
                    EmptyView()
                }
            }
            .keylineTint(Color.accentColor)
        }
    }
}

// MARK: - HR + Secondary metric row

private struct HrAndApRow: View {
    let heartRateTitle: String
    let heartRateValue: String
    let secondaryTitle: String
    let secondaryValue: String
    let secondaryKind: String?
    var valueFont: Font = .title3
    var labelFont: Font = .caption
    var dividerHeight: CGFloat = 34

    private var secondaryIconSystemName: String {
        switch secondaryKind {
        case "distance": return "location.fill"
        default: return "star.fill"
        }
    }

    var body: some View {
        HStack(spacing: 0) {
            IconMetric(iconSystemName: "heart.fill", label: heartRateTitle, value: heartRateValue, valueFont: valueFont, labelFont: labelFont)
                .frame(maxWidth: .infinity, alignment: .leading)
            Rectangle().fill(Color.white.opacity(0.25)).frame(width: 1, height: dividerHeight).padding(.horizontal, 12)
            IconMetric(iconSystemName: secondaryIconSystemName, label: secondaryTitle, value: secondaryValue, valueFont: valueFont, labelFont: labelFont)
                .frame(maxWidth: .infinity, alignment: .leading)
        }
    }
}

private struct IconMetric: View {
    let iconSystemName: String
    let label: String
    let value: String
    let valueFont: Font
    let labelFont: Font

    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack(spacing: 6) {
                Image(systemName: iconSystemName).font(labelFont).foregroundStyle(Color.white.opacity(0.78))
                Text(label).font(labelFont).foregroundStyle(Color.white.opacity(0.72)).lineLimit(1)
            }
            Text(value).font(valueFont).fontWeight(.bold).monospacedDigit().foregroundStyle(.white).lineLimit(1)
        }
    }
}

// MARK: - ContentState Helpers

extension LiveWorkoutAttributes.ContentState {
    fileprivate var heartRateMetricForRow: LiveWorkoutAttributes.Metric? { metrics.first }
    fileprivate var secondaryMetricForRow: LiveWorkoutAttributes.Metric? {
        guard metrics.count > 1 else { return nil }
        return metrics[1]
    }
    fileprivate var secondaryMetricKindForRow: String? { secondaryMetricKind }

    fileprivate var heartRateValueWithoutUnit: String {
        if let hr = heartRateBpm, hr > 0 { return String(hr) }
        let fullValue = heartRateMetricForRow?.value ?? "--"
        let digits = fullValue.components(separatedBy: CharacterSet.decimalDigits.inverted).joined()
        return digits.isEmpty ? "--" : digits
    }

    fileprivate var heartRateTitleForRow: String {
        let title = heartRateMetricForRow?.label.trimmingCharacters(in: .whitespacesAndNewlines)
        return (title?.isEmpty == false) ? title! : "HR"
    }

    fileprivate var secondaryTitleForRow: String {
        let title = secondaryMetricForRow?.label.trimmingCharacters(in: .whitespacesAndNewlines)
        if title?.isEmpty == false { return title! }
        return secondaryMetricKind == "distance" ? "Distance" : "AP"
    }

    fileprivate var heartRateDisplayValueForRow: String {
        let v = heartRateMetricForRow?.value.trimmingCharacters(in: .whitespacesAndNewlines)
        return (v?.isEmpty == false) ? v! : "--"
    }

    fileprivate var secondaryDisplayValueForRow: String {
        let v = secondaryMetricForRow?.value.trimmingCharacters(in: .whitespacesAndNewlines)
        return (v?.isEmpty == false) ? v! : "--"
    }
}

// MARK: - HR Zones Bar

private struct HrZonesBar: View {
    let heartRateBpm: Int?
    let maxHeartRateBpm: Int

    private var progress: Double {
        let maxHr = Double(maxHeartRateBpm)
        let baseline = maxHr * 0.5
        if maxHr <= 0 { return 0 }
        let hr = Double(heartRateBpm ?? Int(baseline))
        return ((hr - baseline) / (maxHr - baseline)).clamped01
    }

    private func zoneFor(hr: Int, maxHr: Int) -> Int {
        guard maxHr > 0 else { return 1 }
        let pct = (Double(hr) / Double(maxHr)).clamped01
        if pct < 0.6 { return 1 }
        if pct < 0.7 { return 2 }
        if pct < 0.8 { return 3 }
        if pct < 0.9 { return 4 }
        return 5
    }

    private func zoneColor(_ zone: Int) -> Color {
        switch zone {
        case 1: return Color.white.opacity(0.28)
        case 2: return Color(hex: 0x1FBEDC)
        case 3: return Color(hex: 0x1FCB71)
        case 4: return Color(hex: 0xF5C01A)
        default: return Color(hex: 0xE5175D)
        }
    }

    var body: some View {
        GeometryReader { geo in
            let w = max(0, geo.size.width)
            let barHeight: CGFloat = 3
            let dotDiameter: CGFloat = 12
            let dotRadius = dotDiameter / 2
            let dotX = (CGFloat(progress) * w).clamped(min: dotRadius, max: max(dotRadius, w - dotRadius))
            let baselineHr = Int(Double(maxHeartRateBpm) * 0.5)
            let zone = zoneFor(hr: heartRateBpm ?? baselineHr, maxHr: maxHeartRateBpm)
            let hasHr = (heartRateBpm ?? 0) > 0

            ZStack(alignment: .leading) {
                HStack(spacing: 2) {
                    ForEach(1...5, id: \.self) { z in
                        Rectangle().fill(zoneColor(z))
                    }
                }
                .frame(height: barHeight)
                .clipShape(RoundedRectangle(cornerRadius: barHeight / 2, style: .continuous))
                .overlay(
                    RoundedRectangle(cornerRadius: barHeight / 2, style: .continuous)
                        .stroke(Color.white.opacity(0.10), lineWidth: 1)
                )
                .frame(maxHeight: .infinity, alignment: .center)

                Circle()
                    .fill(Color.white)
                    .frame(width: dotDiameter, height: dotDiameter)
                    .shadow(color: Color.black.opacity(0.25), radius: 2, x: 0, y: 1)
                    .offset(x: dotX - dotRadius)
                    .opacity(hasHr ? 1 : 0.55)
            }
        }
    }
}

// MARK: - Utility Extensions

private extension Double {
    var clamped01: Double { min(1, max(0, self)) }
}

private extension CGFloat {
    func clamped(min lower: CGFloat, max upper: CGFloat) -> CGFloat {
        Swift.min(upper, Swift.max(lower, self))
    }
}

private extension Color {
    init(hex: UInt32, alpha: Double = 1.0) {
        let r = Double((hex >> 16) & 0xFF) / 255.0
        let g = Double((hex >> 8) & 0xFF) / 255.0
        let b = Double(hex & 0xFF) / 255.0
        self = Color(.sRGB, red: r, green: g, blue: b, opacity: alpha)
    }
}
```

---

**Previous:** [Engine Lifecycle](08-engine-lifecycle.md) | **Next:** [API Reference](10-api-reference.md) | **Home:** [README](README.md)
