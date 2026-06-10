# Permissions Gate (Open Design)

A "permissions gate" is an onboarding screen that blocks the user until the permissions a feature needs (Bluetooth, Location, or both) have been granted. **Whether the Rolla SDK should ship one — and whether it owns gating at all — is an open product decision and is not settled.** This page documents the current behavior and the two options on the table so partners know what to expect; it does **not** recommend one.

> **This is a forward-looking design note, not a stable API.** Nothing here is a contract. The configurable gate described below does not exist in the public `rolla_sdk` surface today. Do not build against it. Track the decision before relying on any of it.

## What ships today

Today the SDK does **not** expose a configurable permissions gate. Instead, **the SDK requests runtime permissions itself, on demand**, when the user reaches a feature that needs them inside `RollaSdkHome` (for example, the Bluetooth dialog appears when band pairing starts). There is no host-facing screen that hard-blocks the experience up front, and there is no public API to declare "Bluetooth is required" vs. "optional."

The full-screen gate that does this blocking-up-front behavior — including per-permission required flags — currently lives in **`wl-rolla-mobile`**, the Rolla white-label app, **not in the SDK package**. It is not exported from `package:rolla_sdk/rolla_sdk.dart`, so partners cannot import or configure it.

> **Don't confuse this with the runtime prompts you already get.** The SDK still asks for permissions when a feature is used — see [Permissions → Runtime prompts](03-permissions.md#runtime-prompts). The open question is only about an *up-front, host-configurable gate*, not about whether permissions are requested at all.

## The two options (both open)

This is pending a Rolla product decision with Živko. Both options are live; neither is chosen.

### Option A — the SDK ships a configurable gate

The SDK would expose a gate widget (or an init flag) that the host configures with per-permission **required** flags, so the partner declares which permissions hard-block the experience:

- Bluetooth required (band pairing is mandatory in this deployment)
- Location required (outdoor tracking is mandatory)
- Both required
- Either optional (gate informs, but lets the user proceed)

This mirrors the gate already in `wl-rolla-mobile`, where each permission group carries a `required` flag. Pros: consistent UX across all partners, one place to maintain. Cons: another configuration surface to design, version, and document; partners with their own onboarding may not want it.

### Option B — the partner owns permission gating

The SDK requests permissions on demand (today's behavior) and the **partner builds any blocking/onboarding screen themselves**, using their own permission plugin (e.g. `permission_handler`) before pushing the Rolla route. Pros: no new SDK API; partners keep full control of onboarding UX. Cons: every partner re-implements the same gate; inconsistent behavior across deployments; easy to ship an app that reaches a feature and gets a runtime prompt mid-flow.

> **No recommendation is given here.** Choosing A or B affects the public API surface and the platform permission docs. Until it is settled, integrate against today's behavior: declare the permission strings yourself (see below) and let the SDK prompt on demand.

## What you must do regardless of the outcome

Independent of which option lands, a pure-Flutter host **must still declare the underlying platform permissions itself** — the gate decision is about *UX flow*, not about *whether the strings exist*. Without the declarations the app SIGABRT-crashes on iOS and fails at runtime on Android.

- iOS: `Info.plist` usage-description strings.
- Android: `AndroidManifest.xml` `<uses-permission>` entries.

These are covered in full, with exact strings and per-permission rationale, in:

- [Flutter Permissions](03-permissions.md) — the Info.plist keys and AndroidManifest entries you add.
- [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md) — exact strings, App Store privacy rationale.
- [Android Permissions](../android/03-permissions.md) — per-permission rationale, Health Connect `<queries>`, Mapbox token.

---

**Next:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
