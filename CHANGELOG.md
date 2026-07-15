# Changelog

**Tags used in this changelog:**

- `[breaking]` — changes that require immediate attention in order to avoid a failed build.
- `[feature]` — new SDK capability.
- `[improvement]` — existing behavior enhanced, optimized or refactored.
- `[documentation]` — documentation-only change.
- `[fix]` — bug fix.

---

## 0.1.12

### Both platforms

- **[improvement] Skip the SDK's onboarding by setting the profile in advance.** Your app can call `POST /api/setprofile` with the user's access token before first presenting the SDK: when the profile already carries a username, birthdate, gender and height — plus weight when the weight module is enabled — the account-details onboarding is skipped entirely. A partially set profile pre-fills the onboarding form so users only complete the gaps. See the new [Profile](sdk-auth-api/03-profile.md) guide in the Auth API section.

- **[feature] New headless public SDK methods.** Four methods run without the SDK UI ever being opened: `warmUpEngine` (start the engine ahead of time so the first `show()` is instant), `syncHealthData` (full sync of the user's primary data source with a typed result — outcome, skip reason, per-stream `syncedData` breakdown, and `startedAt`/`lastSyncAt` timing), `getBandBatteryLevel` (live battery read from the paired Rolla band, or a typed "unavailable" reason), and `getPairedBandInfo` (paired-band query with zero Bluetooth — `paired`/`notPaired`/`unknown`). Headless calls can't prompt, so a missing OS permission fails fast with a typed reason instead. See the [Android](android/08-api-reference.md#headless-methods) / [iOS](ios/10-api-reference.md#headless-methods) API references and the [Android](android/07-engine-lifecycle.md#warming-up-the-engine) / [iOS](ios/08-engine-lifecycle.md#warming-up-the-engine) engine-lifecycle guides.

- **[feature] Host event callbacks: eleven new SDK events plus a headless-sync completion callback.** `RollaDelegate` (iOS) / `RollaListener` (Android) gains methods your app can override to observe what happens inside the SDK without polling: activity lifecycle (started / completed / removed), UI sync completed, band paired / unpaired, band connected / disconnected (live BLE link transitions), primary source changed, goals changed, and profile updated — plus `rollaDidCompleteHealthDataSync` / `onSyncHealthDataCompleted`, which reports every headless `syncHealthData` result. All methods have default no-op bodies, so existing integrations compile unchanged; events are delivered for the engine's lifetime and keep flowing after the SDK UI closes. See the [Android](android/08-api-reference.md#host-events) / [iOS](ios/10-api-reference.md#host-events) API references for the full tables and delivery semantics.

- **[feature] Hide selected data sources from the SDK UI (`disabledDataSources`).** A new `RollaConfiguration` parameter lets you hide specific data-source connect options (band, Garmin, Oura, Apple Health, Health Connect) wherever the user picks a source to connect — the Data Sources screen and the onboarding data-source step. Pass a set of `RollaDataSource` values to hide those sources, or omit it / pass an empty set to offer everything (default, no change for existing integrations). A source the user has already connected stays visible for viewing/disconnecting; if you disable every source the Rolla Band remains available so onboarding never dead-ends. When the band is the only source left, the picker is skipped entirely — onboarding goes straight to band pairing and the Data Sources entry is hidden from Settings. See [Android](android/05-branding-and-modules.md#data-source-configuration) and [iOS](ios/05-branding-and-modules.md#data-source-configuration) branding & modules guides and the [Android](android/08-api-reference.md#rolladatasource) / [iOS](ios/10-api-reference.md#rolladatasource) API references.

### iOS

- **[breaking] `RollaDelegate` error method renamed: `rolla(_:didFailWithError:)` → `rollaDidFailWithError(_:error:)`.** A signature-only change aligning the one anonymous-form method with the rest of the `rollaDid…` delegate family — same parameters, same behavior: `func rollaDidFailWithError(_ rolla: Rolla, error: RollaError)`. See the [iOS API reference](ios/10-api-reference.md#rolladelegate-protocol).

## 0.1.11

### Both platforms

- **[breaking] Module configuration switched from an enable-list to an opt-out list (`disabledModules`).** The previous enable-list parameter — documented as `modules`, named `enabledModules` in the SDK — has been removed and replaced by `disabledModules`. Pass a set of `RollaDisabledModule` values to hide a module's entire UI, or omit it to keep everything enabled. `weight` and `bloodPressure` are the first two modules supported for disabling. Any integration that passed an enable-list must switch to `disabledModules`. See [Android](android/05-branding-and-modules.md#module-configuration) and [iOS](ios/05-branding-and-modules.md#module-configuration) branding & modules guides and the [Android](android/08-api-reference.md#rolladisabledmodule) / [iOS](ios/10-api-reference.md#rolladisabledmodule) API references.

- **[feature] Added the `removeRollaBandReferences` flag to `RollaConfiguration`, default `true`.** When `true` the SDK UI uses generic "fitness device" wording; set it to `false` to show Rolla Band-specific references. See [Android](android/05-branding-and-modules.md#rolla-band-references) and [iOS](ios/05-branding-and-modules.md#rolla-band-references) branding & modules guides.

- **[feature] Smartphone-only workout tracking.** Workouts can now be started and tracked without a paired wearable, using the phone's pedometer and motion sensors. This adds a new permission requirement on each platform — see the Android and iOS notes below.

- **[feature] Added an Activity History screen.** Users can open it from **View All Activities** at the bottom of the activities section on the SDK Home screen, and browse all past workouts in a monthly calendar view with summary stats and shareable card previews.

- **[feature] Added in-app usage events analytics.** The SDK records basic usage events within its UI (screen views and feature interactions) and reports them to the Rolla backend, queued locally and delivered across offline periods.

- **[feature] Manual activity logging.** Users can manually log a past workout — pick an activity type, set duration and intensity — and the SDK estimates calories from heart-rate samples where available, falling back to a metabolic-equivalents (MET) model otherwise.

- **[feature] New activity types: Spa and Calisthenics.** Calisthenics joins the Strength category and can be live-tracked or logged manually; a new Spa category (Sauna, Steam Room, Cold Plunge, Jacuzzi) is available from the manual activity logger only and does not appear in the live-tracking start list.

- **[improvement] Redesigned the Insights experience into a dedicated Insights tab.** Insights moved off the Home screen into its own tab in the SDK bottom navigation, with a daily scrollable feed, filters, full article views, and ratings. This is visible only to partners using the SDK's bottom navigation bar — partners with their own navigation will no longer see the Insights section.

- **[improvement] Redesigned the SDK bottom navigation bar.** The bottom navigation is now a floating pill with a blurred backdrop, an animated active-tab indicator, and a separated circular button for starting workouts; three primary tabs (Home, Insights, Profile). Partners not using the SDK's bottom navigation will only see the Plus button move from bottom-centre to bottom-right.

- **[improvement] Reduced the SDK payload size by 73 MB, removing unused bundled media assets and lowering your app's download size significantly.**

- **[fix] Apple Health and Health Connect now sync as a secondary source.** Workouts, weight, and blood pressure from a secondary Apple Health / Health Connect connection are now uploaded on each Home resume; previously they stopped syncing when another source was primary. Heart rate, HRV, steps, and sleep remain owned by the primary source.

### Android

- **[breaking] Smartphone-only workouts require `ACTIVITY_RECOGNITION` (API 29+).** The SDK's bundled manifest already declares `android.permission.ACTIVITY_RECOGNITION` to read the phone's step counter, so the manifest merger adds it for you — or you can declare it yourself. Make sure your Play Console listing covers the activity-recognition rationale. See [Smartphone-Only Workouts](android/03-permissions.md#smartphone-only-workouts-activity_recognition) and the updated [Permissions Rationale](android/03-permissions.md#permissions-rationale).

### iOS

- **[breaking] `NSMotionUsageDescription` now required in the host app's `Info.plist`.** Smartphone-only workouts use `CMPedometer`, and iOS hard-terminates any app that starts it without an `NSMotionUsageDescription` string declared. Add the key with a user-facing rationale or smartphone-only workouts will crash the app on first start. See [Motion & Fitness](ios/03-permissions-and-entitlements.md#motion--fitness-required-for-smartphone-only-workouts).

- **[improvement] Live Workout widget honors phone-only mode.** `LiveWorkoutAttributes.ContentState` gained an `isPhoneOnly` flag (defaulted to `false`) so the Dynamic Island and Lock Screen views can hide band-specific elements when a workout is tracked from the phone alone. If you copied an older data contract into your widget, add the field to match the current SDK. See [Live Activities](ios/09-live-activities.md#step-3-widget-extension-files).

---

## 0.1.10

### Both platforms

- **[feature] Added the `showSettingsButton` boolean config on `RollaConfiguration`.** Renders a Settings button on the Home screen that opens a sheet with Data Sources and Goals shortcuts. Defaults to `true` since most partners need it automatically. See [Android](android/08-api-reference.md) and [iOS](ios/10-api-reference.md) API references.

- **[improvement] Improved the GPS tracking to be more accurate on iOS and Android.** The location pipeline has been refactored on both platforms to hold steady when you're standing still, filter out GPS zig-zags, and recover cleanly when you start moving again. Additionally, the in-app map views got some general UX improvements and polishing.

- **[documentation] Added a permissions rationale matrix to the iOS and Android permissions docs.** Each permission is grouped by capability (Location, Bluetooth, Health Connect / Apple Health, etc.) and carries a required/optional status plus a partner-ready rationale suitable for App Store and Play Console submissions. See [Android permissions rationale](android/03-permissions.md#permissions-rationale) and [iOS permissions rationale](ios/03-permissions-and-entitlements.md#permissions-rationale).

### Android

- **[breaking] `minSdk` raised from 24 to 26.** Required by the bundled Health Connect plugin's manifest.

- **[breaking] Host-app Kotlin floor raised to 2.2.0.** Required by the bundled `health` plugin's transitive `kotlin-stdlib-jdk7:2.2.10`. Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum.

- **[feature] Google Health Connect support.** Android health data can now be synced and tracked from Health Connect. For more information about permissions, please see [Health Connect](android/03-permissions.md#health-connect-android).

- **[improvement] Build JDK floor lowered from 21 to 17.** AAR is now compiled with Java 17 (class file major version 61).

### iOS

- **[improvement] Simulator support added (Debug configuration).** `0.1.10` runs on iPhone simulators under the Debug configuration, in addition to the existing Release-on-device support. Hardware-backed features (Bluetooth, etc.) still only work on physical devices. See [iOS Prerequisites](ios/01-prerequisites.md).

---

## 0.1.9 — First stable release

`0.1.9` is the first stable release of the Rolla SDK. Per-change entries begin at `0.1.10`.
