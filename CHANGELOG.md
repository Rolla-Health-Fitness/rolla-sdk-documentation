# Changelog

**Tags used in this changelog:**

- `[breaking]` — changes that require immediate attention in order to avoid a failed build.
- `[feature]` — new SDK capability.
- `[improvement]` — existing behavior enhanced, optimized or refactored.
- `[documentation]` — documentation-only change.
- `[fix]` — bug fix.

---

## 0.1.14

### Android

- **[breaking] The public API types moved into sub-packages — update your imports.** No type was renamed and no behavior changed, so the fix is import lines only; most IDEs re-import automatically. `Rolla` and `RollaListener` are unchanged in `com.rolla.sdk.wrapper`. Everything else moved: `RollaConfiguration`, `RollaBranding`, `RollaLanguage`, `RollaThemeMode`, `RollaTransition`, `RollaDataSource` and `RollaDisabledModule` to `com.rolla.sdk.wrapper.config`; `RollaError` and `RollaCloseReason` to `…features.session`; the activity payloads to `…features.activity`; the band payloads to `…features.band`; `RollaSyncResult` and `RollaPrimarySourceChanged` to `…features.sync`; `RollaGoalsChanged` to `…features.goals`; `RollaProfileUpdated` to `…features.profile`. Enums travel with the file that declares them — `RollaSyncOutcome` is in `…features.sync`, `RollaBatteryStatus` in `…features.band`. If you declare the SDK activity in your own manifest, it is now `com.rolla.sdk.wrapper.engine.RollaFlutterActivity`.

### Both platforms

- **[breaking] `showSettingsButton` is renamed to `showOptionsButton`, and the entry moved to the top right app bar actions.** The Settings button that was positioned at the very bottom of the Home screen is now a three-dot options action at the trailing edge of the Home app bar, visible without scrolling. It opens the same bottom sheet of shortcuts as before, now titled "Options". The flag's meaning and default (`true`) are unchanged — just rename the parameter on your `RollaConfiguration`. See the [Android](android/05-configuration.md#rollaconfiguration) and [iOS](ios/05-configuration.md#rollaconfiguration) configuration guides.

- **[breaking] Removed the per-module configuration API, which never had any effect.** The `moduleConfigs` parameter of `initializeWithToken`, the `RollaModuleConfig` class and all of its `<Module>ModuleConfig` subclasses, and `RollaSDKConfig.moduleConfigs` / `getModuleConfig` are gone. Every value they carried was ignored by the SDK, so removing them changes no behavior — but code that constructed these objects no longer compiles. Delete the `moduleConfigs` argument and any `<Module>ModuleConfig` instances; to hide a module's UI use `disabledModules` instead. See the [Android](android/05-configuration.md#rolladisabledmodule) / [iOS](ios/05-configuration.md#rolladisabledmodule) configuration guides.

- **[breaking] Removed the `RollaNavigationDelegate` and `RollaStorageProvider` integration points, which were never implemented.** Along with the `navigationDelegate` and `storageProvider` fields of `RollaSDKConfig`. The SDK never read them, so no behavior changes; remove any references.

- **[feature] Open a specific SDK screen from your app.** The new `openScreen` method opens the Insights feed, the activity history, the goals editor, the SDK Home screen, or the last-opened state (`resume`) directly — presenting the SDK UI first when it is not on screen, honoring the optional `RollaTransition`. The opened screen is the root of the SDK UI, so back returns the user straight to your app, and `home` restores the regular Home entry point without an engine restart. Every outcome is a typed `RollaOpenScreenStatus`; the SDK's mandatory startup steps (onboarding, consent) always take precedence. See the [Android](android/08-api-reference.md#host-driven-navigation) / [iOS](ios/10-api-reference.md#host-driven-navigation) API references.

- **[fix] Leaderboard messages now follow the selected language.** The notice shown after leaving a leaderboard, which explains that rejoining is not possible for 7 days, along with the leaderboard error messages, always appeared in English regardless of the app language.

- **[documentation] Rewritten Token Management guides.** The [Android](android/06-token-management.md) and [iOS](ios/07-token-management.md) pages now spell out the host app's token obligations — persist rotated tokens, answer the token-expired callback, always initialize with the latest pair — along with token lifetimes, the single-use refresh-token rotation rule, and a symptoms table for diagnosing 401 errors.

## 0.1.13

### Both platforms

- **[feature] Insights entry on the Home screen.** A new Insights entry card in the Home Overview section shows the unread insights count and opens the insights feed page. This option can be disabled alongside all other insights UI by adding `RollaDisabledModule.insights` value to the `disabledModules`. See the [Android](android/05-configuration.md#rolladisabledmodule) / [iOS](ios/05-configuration.md#rolladisabledmodule) configuration guides.

- **[feature] Optional Goals section on Home via the new `showGoalsSection` configuration flag.** `RollaConfiguration` gains an optional `showGoalsSection` (default `false`). When `true`, the bottom of the Home screen shows the user's enabled goals with an edit action — or a select-goals call-to-action when zero goals are selected. See the [Android](android/05-configuration.md#goals-on-home) / [iOS](ios/05-configuration.md#goals-on-home) configuration guides.

- **[feature] New `RollaTransition` animation on the `show()` method.** A new optional `transition` parameter controls how the SDK UI opens and closes: `.default` is the existing animation, `.fade` is a cross-fade. The closing transition always mirrors the opening one. See the [Android](android/08-api-reference.md#rollatransition) / [iOS](ios/10-api-reference.md#rollatransition) API references.

- **[fix] Confirmation before changing the primary data source.** Switching your primary data source now asks for confirmation first, so it can no longer happen from an accidental tap.

- **[improvement] Refined Serbian translations.** Both Serbian scripts — Latin and Cyrillic — received a native-speaker terminology pass across the entire SDK UI.

- **[improvement] General bugfixes and stability improvements.**

## 0.1.12

### Both platforms

- **[feature] New headless public SDK methods.** Four methods run **headlessly** — no SDK UI needs to be opened:
  - **`warmUpEngine()`** — start the engine ahead of time so the first `show()` presents instantly.
  - **`syncHealthData()`** — full sync of the user's primary data source, returning a typed result: outcome, skip reason, per-stream `syncedData` breakdown, and `startedAt`/`lastSyncAt` timing.
  - **`getBandBatteryLevel()`** — live battery read from the paired Rolla band, or a typed "unavailable" reason.
  - **`getPairedBandInfo()`** — paired-band query with zero Bluetooth: `bandPaired`/`noBandPaired`/`unknown`.

  See the [Android](android/08-api-reference.md#headless-methods) / [iOS](ios/10-api-reference.md#headless-methods) API references and the [Android](android/07-engine-lifecycle.md#warming-up-the-engine) / [iOS](ios/08-engine-lifecycle.md#warming-up-the-engine) engine-lifecycle guides.

- **[feature] Host event callbacks: twelve new delegate/listener methods.** `RollaDelegate` (iOS) / `RollaListener` (Android) gains methods your app can override to observe the SDK without polling — all with default no-op bodies, delivered for the engine's lifetime (they keep flowing after the SDK UI closes):

  | Event | iOS | Android |
  |-------|-----|---------|
  | Activity&nbsp;started | `rollaDidStartActivity` | `onActivityStarted` |
  | Activity&nbsp;completed | `rollaDidCompleteActivity` | `onActivityCompleted` |
  | Activity&nbsp;removed | `rollaDidRemoveActivity` | `onActivityRemoved` |
  | UI&nbsp;sync&nbsp;completed | `rollaDidCompleteUISync` | `onUiSyncCompleted` |
  | Headless&nbsp;sync&nbsp;completed | `rollaDidCompleteHealthDataSync` | `onSyncHealthDataCompleted` |
  | Band&nbsp;paired | `rollaDidPairBand` | `onBandPaired` |
  | Band&nbsp;unpaired | `rollaDidUnpairBand` | `onBandUnpaired` |
  | Band&nbsp;connected | `rollaDidConnectBand` | `onBandConnected` |
  | Band&nbsp;disconnected | `rollaDidDisconnectBand` | `onBandDisconnected` |
  | Primary&nbsp;source&nbsp;changed | `rollaDidChangePrimarySource` | `onPrimarySourceChanged` |
  | Goals&nbsp;changed | `rollaDidChangeGoals` | `onGoalsChanged` |
  | Profile&nbsp;updated | `rollaDidUpdateProfile` | `onProfileUpdated` |

  See the [Android](android/08-api-reference.md#host-events) / [iOS](ios/10-api-reference.md#host-events) Host Events sections for the payloads and delivery semantics.

- **[feature] Host-controlled SDK language (`language` on `RollaConfiguration`).** Typed by the new `RollaLanguage` enum; when set it is authoritative for the engine's lifetime — persisted in-SDK picks and the backend profile can't override it — and is written to the user's backend profile at startup so backend-generated content (goal labels, insights) matches the UI language. Unset keeps the profile-driven behavior. The SDK now ships eight languages: English, German, **Spanish (new)**, Croatian, Bosnian, **Serbian — Latin and Cyrillic (new)**, and Arabic. See the [Android](android/05-configuration.md#language) / [iOS](ios/05-configuration.md#language) configuration guides.

- **[feature] New Leaderboards module — and a `leaderboards` value in `RollaDisabledModule` to hide it.** Opt-in weekly/monthly rankings comparing users in your tenant on Health Score or Active Points, with join/leave controls. Enabled by default; pass `RollaDisabledModule.leaderboards` in `disabledModules` to hide it everywhere in the SDK UI. See the [Android](android/05-configuration.md#rolladisabledmodule) / [iOS](ios/05-configuration.md#rolladisabledmodule) configuration guides.

- **[feature] Hide selected data sources from the SDK UI (`disabledDataSources`).** A new `RollaConfiguration` parameter that hides specific connect options (band, Garmin, Oura, Apple Health, Health Connect) wherever the user picks a source. Deny-list semantics: omit it or pass an empty set to offer everything; already-connected sources stay visible for viewing/disconnecting; disabling every source keeps the Rolla Band as a floor, and when the band is the only source left the picker is skipped — onboarding goes straight to pairing. See the [Android](android/05-configuration.md#data-source-configuration) / [iOS](ios/05-configuration.md#data-source-configuration) configuration guides.

- **[breaking] `RollaBranding` reworked to hold exactly the options that affect the SDK.** Six fields, all optional: `hostAppName`, `primaryColor`, `themeMode` (renamed from `defaultThemeMode`, typed by the new `RollaThemeMode` enum), `headerLogoAsset`, `privacyUrl`, and `removeRollaBandReferences` (moved from `RollaConfiguration`, same semantics). A set field overrides the SDK default individually; an unset field keeps it — previously, passing any branding replaced all defaults at once. The removed options — `appName`, `secondaryColor`, `accentColor`, `brightness`, `defaultLocale`, `termsUrl` — had no effect on the SDK UI. See the [Android](android/05-configuration.md#custom-branding-optional) / [iOS](ios/05-configuration.md#custom-branding-optional) configuration guides.

- **[improvement] Skip the SDK's onboarding by setting the profile in advance.** Call `POST /api/setprofile` before first presenting the SDK: a profile carrying username, birthdate, gender, height, and weight skips the account-details onboarding entirely; a partial profile pre-fills the form. Weight is always required — the SDK uses it to calculate calories, even with the weight module disabled. See the new [Profile](sdk-auth-api/03-profile.md) guide.

- **[improvement] Split the combined permission screen into separate Bluetooth and Location pages.** Each permission now has its own page with contextual copy explaining why it is needed.

- **[fix] Bugs and stability fixes.** Various internal fixes and stability improvements.

- **[documentation] Updated and restructured the documentation for this release's many new features and breaking changes.** The *Branding & Modules* pages became the per-platform [Configuration](ios/05-configuration.md) guides ([Android](android/05-configuration.md)) covering every `RollaConfiguration` option, and the `RollaConfiguration` reference moved there from the API references.

### Android

- **[improvement] Brand-neutral notification channel names.** The two SDK-created notification channels end users see in system settings were renamed from "Rolla Warnings" and "Engagement" to "Important Alerts" and "Engagement Tips", so they read naturally under the host app's branding. A new [Notification Channels](android/03-permissions.md#notification-channels) section documents every channel the SDK creates.

### iOS

- **[breaking] `RollaDelegate` error method renamed: `rolla(_:didFailWithError:)` → `rollaDidFailWithError(_:error:)`.** A signature-only change aligning the one anonymous-form method with the rest of the `rollaDid…` delegate family — same parameters, same behavior: `func rollaDidFailWithError(_ rolla: Rolla, error: RollaError)`. See the [iOS API reference](ios/10-api-reference.md#rolladelegate-protocol).

## 0.1.11

### Both platforms

- **[breaking] Module configuration switched from an enable-list to an opt-out list (`disabledModules`).** The previous enable-list parameter — documented as `modules`, named `enabledModules` in the SDK — has been removed and replaced by `disabledModules`. Pass a set of `RollaDisabledModule` values to hide a module's entire UI, or omit it to keep everything enabled. `weight` and `bloodPressure` are the first two modules supported for disabling. Any integration that passed an enable-list must switch to `disabledModules`. See the [Android](android/05-configuration.md#module-configuration) and [iOS](ios/05-configuration.md#module-configuration) configuration guides and the [Android](android/05-configuration.md#rolladisabledmodule) / [iOS](ios/05-configuration.md#rolladisabledmodule) `RollaDisabledModule` values.

- **[feature] Added the `removeRollaBandReferences` flag to `RollaConfiguration`, default `true`.** When `true` the SDK UI uses generic "fitness device" wording; set it to `false` to show Rolla Band-specific references. See the [Android](android/05-configuration.md#rolla-band-references) and [iOS](ios/05-configuration.md#rolla-band-references) configuration guides.

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

- **[feature] Added the `showSettingsButton` boolean config on `RollaConfiguration`.** Renders a Settings button on the Home screen that opens a sheet with Data Sources and Goals shortcuts. Defaults to `true` since most partners need it automatically. See the [Android](android/05-configuration.md#rollaconfiguration) and [iOS](ios/05-configuration.md#rollaconfiguration) configuration guides.

- **[improvement] Improved the GPS tracking to be more accurate on iOS and Android.** The location pipeline has been refactored on both platforms to hold steady when you're standing still, filter out GPS zig-zags, and recover cleanly when you start moving again. Additionally, the in-app map views got some general UX improvements and polishing.

- **[documentation] Added a permissions rationale matrix to the iOS and Android permissions docs.** Each permission is grouped by capability (Location, Bluetooth, Health Connect / Apple Health, etc.) and carries a required/optional status plus a partner-ready rationale suitable for App Store and Play Console submissions. See [Android permissions rationale](android/03-permissions.md#permissions-rationale) and [iOS permissions rationale](ios/03-permissions-and-entitlements.md#permissions-rationale).

### Android

- **[breaking] `minSdk` raised from 24 to 26.** Required by the bundled Health Connect plugin's manifest.

- **[breaking] Host-app Kotlin floor raised to 2.2.0.** Required by the bundled `health` plugin's transitive `kotlin-stdlib-jdk7:2.2.10`. Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum.

- **[feature] Google Health Connect support.** Android health data can now be synced and tracked from Health Connect. For more information about permissions, please see [Health Connect](android/03-permissions.md#health-connect-android).

- **[improvement] Build JDK floor lowered from 21 to 17.** AAR is now compiled with Java 17 (class file major version 61).

### iOS

- **[improvement] Simulator support added (Debug configuration).** `0.1.10` runs on iPhone simulators under the Debug configuration, in addition to the existing Release-on-device support. Hardware-backed features (Bluetooth, etc.) still only work on physical devices. See the [iOS Quick Start](ios/00-quick-start.md).

---

## 0.1.9 — First stable release

`0.1.9` is the first stable release of the Rolla SDK. Per-change entries begin at `0.1.10`.
