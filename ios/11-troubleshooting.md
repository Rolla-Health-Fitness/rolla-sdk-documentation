# Troubleshooting

This section provides solutions to common issues encountered when integrating and using the Rolla SDK.

## Common Issues

### SDK fails to start

- Ensure all Info.plist permissions are configured
- Verify Bluetooth Central capability is enabled
- Check that the token is valid and not expired
- Verify HealthKit capability is enabled (if using Apple Health)

### Apple Health not showing

- Verify `NSHealthShareUsageDescription` is in Info.plist
- Verify HealthKit capability is added in Xcode (Signing & Capabilities)
- HealthKit is not available on iPads without the Health app

### Build errors

- Clean build folder: Product > Clean Build Folder (Shift-Command-K)
- Delete Pods folder and Podfile.lock
- Run `pod install` again
- Ensure you're opening `.xcworkspace`, not `.xcodeproj`

### Calling `show()` while the SDK is already presenting

If you call `show(from:)` while the SDK UI is already on screen, the SDK does **not** crash or queue a second presentation. Instead, it immediately fires a `.alreadyPresenting` error through your delegate:

```swift
func rolla(_ rolla: Rolla, didFailWithError error: RollaError) {
    if case .alreadyPresenting = error {
        // SDK is already showing — no action needed
    }
}
```

To avoid this, check `rolla.isPresenting` before calling `show(from:)`, or guard your UI so the user cannot trigger it twice.

### Token-related issues

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| SDK opens then immediately closes or shows an error | Expired access token | Fetch a fresh token from your backend before calling `show(from:)` |
| `rollaDidRequestTokenRefresh` fires immediately | Token was already expired at launch | Ensure `tokenExpiresIn` reflects the *remaining* lifetime, not the original TTL |
| SDK works in `"rnd"` but fails in `"production"` | Token was issued for the wrong environment | Verify your backend issues tokens against the correct Rolla environment |
| "Unauthorized" or 401-style errors | Wrong partner ID or mismatched credentials | Double-check the `partnerId` passed to `RollaConfiguration` |

### Flutter engine crash recovery

If the Flutter engine crashes (rare, but possible under extreme memory pressure), the SDK UI will close and `rollaDidClose` will fire. To recover:

1. Call `Rolla.destroyEngine()` to fully tear down the crashed engine.
2. Create a new `Rolla` instance with a fresh configuration.
3. Call `show(from:)` to restart.

```swift
func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
    Rolla.destroyEngine()
    // Re-create and show when ready
}
```

See [Engine Lifecycle](08-engine-lifecycle.md) for more on `destroyEngine()`.

### Getting debug logs for support tickets

The SDK logs internally using Apple's unified logging system with the subsystem `app.rolla.rollaV2`. To capture these logs:

1. Open **Console.app** on your Mac with the device connected.
2. Filter by subsystem: `app.rolla.rollaV2`.
3. Reproduce the issue.
4. Export the filtered log and attach it to your support ticket.

You can also stream logs from the command line:

```bash
log stream --device --predicate 'subsystem == "app.rolla.rollaV2"' --level info
```

### CocoaPods issues

**`pod install` fails or resolves the wrong version:**

```bash
# Clear the CocoaPods cache and re-install
pod cache clean --all
rm -rf Pods Podfile.lock
pod install
```

**Spec repo out of date:**

```bash
pod repo update
pod install
```

**Use `pod install` vs `pod update`:**
- `pod install` — Use when adding the SDK for the first time or when checking out a project. Respects `Podfile.lock`.
- `pod update RollaSDK` — Use when you want to pull a newer SDK version. Updates the lock file.

**"Unable to find a specification" error:**
- Verify the Rolla SDK source URL in your `Podfile` is correct.
- Ensure you have network access to the Rolla SDK repository.

## Support

For issues or questions, contact Rolla support or refer to the SDK documentation.

---

**Previous:** [API Reference](10-api-reference.md) | **Home:** [README](README.md)
