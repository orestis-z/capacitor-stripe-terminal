# Migration Guide: Upgrading to Capacitor v8 and Stripe Terminal SDK v5

This guide covers the breaking changes when upgrading from Capacitor v4 with Stripe Terminal SDK v2 to Capacitor v8 with Stripe Terminal SDK v5.

## Overview

This update includes:

- **Capacitor**: v4.0.0 → v8.0.2
- **Stripe Terminal SDK (iOS)**: v2.17.1 → v5.2.0
- **Stripe Terminal SDK (Android)**: v2.17.1 → v5.2.0
- **@stripe/terminal-js**: v0.11.0 → v0.26.0

## Requirements

### iOS

- **Minimum iOS version**: 15.0 (increased from 13.0)
- **Xcode**: 16.0+ with Swift 6.2
- **Platform**: iOS 15.0+

### Android

- **Minimum SDK**: API 23 (Android 6.0)
- **Target SDK**: API 34 (Android 14)
- **Gradle**: 8.1.1+

### Node.js & TypeScript

- **Node.js**: 14.x or higher
- **TypeScript**: 4.7+
- **Capacitor**: 8.x

## Breaking Changes

### 1. iOS Minimum Deployment Target

The minimum iOS deployment target has been increased from 13.0 to 15.0.

**Action Required**: Update your app's iOS deployment target in Xcode to 15.0 or higher.

### 2. New Connection Status: Reconnecting

A new connection status `Reconnecting` has been added to the `ConnectionStatus` enum.

```typescript
export enum ConnectionStatus {
  NotConnected = 0,
  Connected = 1,
  Connecting = 2,
  Reconnecting = 3 // NEW in v5
}
```

**Action Required**: If you handle connection status changes, ensure your code handles the new `Reconnecting` status.

```typescript
terminal.connectionStatus().subscribe(status => {
  switch (status) {
    case ConnectionStatus.NotConnected:
      // Handle not connected
      break
    case ConnectionStatus.Connected:
      // Handle connected
      break
    case ConnectionStatus.Connecting:
      // Handle connecting
      break
    case ConnectionStatus.Reconnecting: // NEW
      // Handle auto-reconnect in progress
      break
  }
})
```

### 3. Capacitor v8 API Changes

Capacitor v8 includes some internal changes to plugin APIs, but the public API of this plugin remains the same. No changes are required in your application code related to Capacitor v8.

## No Breaking Changes to Plugin API

**Good news!** The public API of `capacitor-stripe-terminal` remains unchanged. All your existing code using the plugin should continue to work without modifications.

## Installation

1. Update your dependencies:

```bash
npm install capacitor-stripe-terminal@latest
npm install @capacitor/core@8 @capacitor/ios@8 @capacitor/android@8
```

2. Sync native files:

```bash
npx cap sync
```

3. Update iOS deployment target in Xcode:

   - Open your project in Xcode
   - Select your project in the navigator
   - Select your app target
   - Under "Deployment Info", set "Minimum Deployments" to iOS 15.0

4. Clean and rebuild your project:

```bash
# iOS
npx cap open ios
# Then clean build folder in Xcode (Cmd+Shift+K) and rebuild

# Android
npx cap open android
# Then clean and rebuild in Android Studio
```

## Testing Your Migration

After upgrading, test the following scenarios:

1. **Reader Discovery**: Ensure you can discover readers
2. **Reader Connection**: Test connecting to different reader types
3. **Payment Collection**: Verify payment collection works
4. **Reader Disconnection**: Test disconnection and auto-reconnection
5. **Connection Status**: Verify connection status updates work correctly

## Additional Resources

- [Stripe Terminal iOS SDK v5 Migration Guide](https://docs.stripe.com/terminal/references/sdk-migration-guide?terminal-sdk-platform=ios)
- [Stripe Terminal Android SDK v5 Changelog](https://github.com/stripe/stripe-terminal-android/blob/master/CHANGELOG.md)
- [Capacitor v8 Update Guide](https://capacitorjs.com/docs/updating/8-0)

## Getting Help

If you encounter issues during migration:

1. Check the [GitHub Issues](https://github.com/eventOneHQ/capacitor-stripe-terminal/issues)
2. Review the [Stripe Terminal documentation](https://stripe.com/docs/terminal)
3. Open a new issue with details about your problem

## Summary

This upgrade primarily updates the underlying SDKs while maintaining API compatibility. The main action required is updating your iOS deployment target to 15.0 and handling the new `Reconnecting` connection status if you monitor connection state changes.
