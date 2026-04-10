# CocoaPods Setup

This section covers adding the Rolla SDK to your project via CocoaPods, configuring build settings, and installing dependencies.

> **Package Manager Support:** CocoaPods is the only supported package manager at this time. Swift Package Manager (SPM) is not currently supported.

## Update Podfile

Add the Rolla SDK specs repository and dependency to your Podfile:

```ruby
platform :ios, '14.0'

# Add Rolla SDK specs repository
source 'https://github.com/Rolla-Health-Fitness/rolla-sdk-release-ios.git'
source 'https://cdn.cocoapods.org/'

target 'YourApp' do
  use_frameworks!

  # Add Rolla SDK
  pod 'RollaSDK', '~> 0.1.6'
end
```

## Set ENABLE_USER_SCRIPT_SANDBOXING to "No"

Disable `ENABLE_USER_SCRIPT_SANDBOXING`:

1. Open your project in Xcode.
2. In the Project Navigator, click the project (top blue icon).
3. Select your target.
4. Open the Build Settings tab.
5. In the search bar, type: `User Script Sandboxing`
6. You'll see "User Script Sandboxing" under "Build Options".
7. Double-click the value and change it to "No".

> **Why?** CocoaPods needs to create temporary files during framework copying, and sandboxing blocks that.

## Install Dependencies

Run:

```bash
pod install
```

> **Important:** Always open your project using the `.xcworkspace` file, not `.xcodeproj`.

---

**Previous:** [Prerequisites](01-prerequisites.md) | **Next:** [Permissions & Entitlements](03-permissions-and-entitlements.md) | **Home:** [README](README.md)
