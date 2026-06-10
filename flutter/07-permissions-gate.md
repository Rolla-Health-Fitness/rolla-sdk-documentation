# Permission Gating

How and when the SDK requests runtime permissions, and what to do if your onboarding needs more control.

## Current behavior

The SDK requests runtime permissions **on demand**: the relevant prompt (Bluetooth, Location, Motion, Health) appears when the user reaches a feature that needs it inside `RollaSdkHome` — for example, the Bluetooth dialog appears when band pairing starts. Your Flutter code calls no permission API, and there is no host-facing screen that blocks the experience up front.

## Pre-prompting in your own onboarding

If your UX prefers to collect permissions before launching the SDK, build that screen in your app and request the permissions yourself (e.g. with [`permission_handler`](https://pub.dev/packages/permission_handler)) before pushing the Rolla route. Permissions already granted are not re-prompted — the SDK proceeds directly.

This is optional. The SDK works without any up-front gating.

## Declarations are always required

Whatever flow you choose, the underlying platform declarations must exist — gating affects *when* the user is asked, not *whether the strings exist*:

- iOS: `Info.plist` usage-description strings — [Permissions → iOS](03-permissions.md#ios)
- Android: `AndroidManifest.xml` `<uses-permission>` entries — [Permissions → Android](03-permissions.md#android)

---

**Next:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
