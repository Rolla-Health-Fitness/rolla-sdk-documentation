# Troubleshooting

This section provides solutions to common issues encountered when integrating and using the Rolla SDK.

## 14. Troubleshooting

### SDK fails to start

- Ensure all Info.plist permissions are configured
- Verify Bluetooth Central capability is enabled
- Check that the token is valid and not expired
- Verify HealthKit capability is enabled (if using Apple Health)

### Apple Health not showing

- Verify `NSHealthShareUsageDescription` is in Info.plist
- Verify HealthKit capability is added in Xcode (Signing & Capabilities)
- Ensure `"integrations"` module is enabled (or modules is `nil`)
- HealthKit is not available on iPads without the Health app

### Build errors

- Clean build folder: Product > Clean Build Folder (Shift-Command-K)
- Delete Pods folder and Podfile.lock
- Run `pod install` again
- Ensure you're opening `.xcworkspace`, not `.xcodeproj`

## 15. Support

For issues or questions, contact Rolla support or refer to the SDK documentation.

---

**Previous:** [API Reference](10-api-reference.md) | **Home:** [README](README.md)
