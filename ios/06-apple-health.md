# Apple Health Integration

The Rolla SDK includes a full Apple Health integration for reading HealthKit data. This section covers required setup, supported data types, availability, and platform notes.

## 7. Apple Health Integration

The SDK includes a full Apple Health integration. Partners do **not** need to write any HealthKit code — the SDK handles all reading automatically. However, your app must be configured correctly for HealthKit access.

### 7.1 Required Setup

Make sure you have completed:

1. **Info.plist entries** — `NSHealthShareUsageDescription` (see [Permissions & Entitlements](03-permissions-and-entitlements.md))
2. **HealthKit capability** — Added via Xcode Signing & Capabilities (see [Permissions & Entitlements](03-permissions-and-entitlements.md))

No additional code is required. When the user opens the SDK and navigates to the integrations section, the SDK will prompt the user for Apple Health permissions automatically.

### 7.2 Supported Health Data Types

The SDK reads the following 14 HealthKit data types:

| Data Type | Read | Write |
|-----------|:----:|:-----:|
| Steps | Yes | No |
| Heart Rate | Yes | No |
| Heart Rate Variability (HRV) | Yes | No |
| Active Energy Burned | Yes | No |
| Sleep Analysis (with stages: awake, light, deep, REM) | Yes | No |
| Workouts | Yes | No |
| Workout Routes | Yes | No |
| Distance Walking/Running | Yes | No |
| Distance Cycling | Yes | No |
| Cycling Cadence (iOS 17+) | Yes | No |
| Cycling Power (iOS 17+) | Yes | No |
| Running Speed (iOS 16+) | Yes | No |
| Weight (maps to HealthKit Body Mass) | Yes | No |
| Blood Pressure (systolic + diastolic) | Yes | No |

> **Note:** Resting Heart Rate is available as a computed metric within the SDK's metrics module, but it is not read directly from HealthKit.

### 7.3 Apple Health Availability

Apple Health is part of the `integrations` module, which is enabled by default. The SDK gracefully handles devices without HealthKit support (e.g., iPads without the Health app) — on those devices, Apple Health features simply won't appear.

### 7.4 No Background Delivery

Apple Health data is read on-demand, not via background delivery in the current SDK implementation. No additional background modes are required beyond what is already configured (location + bluetooth-central).

### 7.5 Android Equivalent

There is currently **no** Android Health Connect integration in the SDK. On Android, health data comes from the Rolla band only. Do not expect feature parity between platforms for health data integrations.

---

**Previous:** [Branding & Modules](05-branding-and-modules.md) | **Next:** [Token Management](07-token-management.md) | **Home:** [README](README.md)
