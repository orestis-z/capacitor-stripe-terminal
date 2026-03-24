# Migration Guide: Upgrading to Capacitor v8 and Stripe Terminal SDK v5

This guide covers the breaking changes when upgrading from Capacitor v4 with Stripe Terminal SDK v2 to Capacitor v8 with Stripe Terminal SDK v5.

## Overview

This update includes:

- **Capacitor**: v4.0.0 → v8.0.2
- **Stripe Terminal SDK (iOS)**: v2.17.1 → v5.3.0
- **Stripe Terminal SDK (Android)**: v2.17.1 → v5.3.0
- **@stripe/terminal-js**: v0.11.0 → v0.26.0

## Requirements

### iOS

- **Minimum iOS version**: 15.0 (increased from 13.0)
- **Xcode**: 16.0+ with Swift 6.2
- **Platform**: iOS 15.0+

### Android

- **Minimum SDK**: API 23 (Android 6.0)
- **Target SDK**: API 35 (Android 15)
- **Gradle**: 8.3.2+

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
  Reconnecting = 3, // NEW in v5
}
```

**Action Required**: If you handle connection status changes, ensure your code handles the new `Reconnecting` status.

```typescript
const handle = await terminal.connectionStatus((status) => {
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

### 3. `Embedded` and `Handoff` Replaced by `AppsOnDevices` (Android)

The `Embedded` and `Handoff` discovery methods have been removed and replaced by `DiscoveryMethod.AppsOnDevices`, which is the Stripe Terminal Android SDK v5's unified replacement for apps running directly on a reader device (e.g. Stripe Reader S700). This method is Android-only.

`connectHandoffReader` has been replaced by `connectAppsOnDevicesReader`, which uses `ConnectionConfiguration.AppsOnDevicesConnectionConfiguration` from the Stripe Terminal Android SDK v5.

| Before                           | After                                  |
| -------------------------------- | -------------------------------------- |
| `DiscoveryMethod.Embedded`       | `DiscoveryMethod.AppsOnDevices`        |
| `DiscoveryMethod.Handoff`        | `DiscoveryMethod.AppsOnDevices`        |
| `HandoffConnectionConfiguration` | `AppsOnDevicesConnectionConfiguration` |
| `connectHandoffReader()`         | `connectAppsOnDevicesReader()`         |

```typescript
// Before
discoverReaders({ discoveryMethod: DiscoveryMethod.Embedded })
discoverReaders({ discoveryMethod: DiscoveryMethod.Handoff })
connectHandoffReader(reader, config as HandoffConnectionConfiguration)

// After
discoverReaders({ discoveryMethod: DiscoveryMethod.AppsOnDevices })
connectAppsOnDevicesReader(reader)
```

**Action Required**: Replace any usage of `DiscoveryMethod.Embedded` or `DiscoveryMethod.Handoff` with `DiscoveryMethod.AppsOnDevices`. Replace `connectHandoffReader` calls with `connectAppsOnDevicesReader`.

### 4. `VerifoneP400` Device Type Removed (Android)

The `DeviceType.VerifoneP400` enum value has been removed from the Android Stripe Terminal SDK v5. On Android, Verifone P400 readers will now be reported as `DeviceType.Unknown`.

**Action Required**: If your app handles `DeviceType.VerifoneP400` specifically on Android, update your code to also handle `DeviceType.Unknown` for any Verifone P400 devices.

### 5. `LocalMobile` Renamed to `TapToPay`

To align with the Stripe Terminal SDK naming, all `LocalMobile` references have been renamed to `TapToPay`.

| Before                                           | After                                         |
| ------------------------------------------------ | --------------------------------------------- |
| `DiscoveryMethod.LocalMobile`                    | `DiscoveryMethod.TapToPay`                    |
| `LocalMobileConnectionConfiguration`             | `TapToPayConnectionConfiguration`             |
| `connectLocalMobileReader()`                     | `connectTapToPayReader()`                     |
| event `localMobileReaderDidAcceptTermsOfService` | event `tapToPayReaderDidAcceptTermsOfService` |

**Action Required**: Update all usages in your code:

```typescript
// Before
import {
  DiscoveryMethod,
  LocalMobileConnectionConfiguration,
} from 'capacitor-stripe-terminal'

terminal.discoverReaders({ discoveryMethod: DiscoveryMethod.LocalMobile })
terminal.connectLocalMobileReader(
  reader,
  config as LocalMobileConnectionConfiguration,
)
terminal.addListener('localMobileReaderDidAcceptTermsOfService', handler)

// After
import {
  DiscoveryMethod,
  TapToPayConnectionConfiguration,
} from 'capacitor-stripe-terminal'

terminal.discoverReaders({ discoveryMethod: DiscoveryMethod.TapToPay })
terminal.connectTapToPayReader(
  reader,
  config as TapToPayConnectionConfiguration,
)
terminal.addListener('tapToPayReaderDidAcceptTermsOfService', handler)
```

### 6. `AppleBuiltIn` Renamed to `TapToPay` in `DeviceType`; Android Support Added

`DeviceType.AppleBuiltIn` has been renamed to `DeviceType.TapToPay` to reflect that this device type now covers both the iOS Tap to Pay reader and the Android Tap to Pay device. The numeric value (`11`) is unchanged.

New device types have also been added for the Stripe S700 DevKit, S710, and S710 DevKit readers:

| Added                                      | Value |
| ------------------------------------------ | ----- |
| `DeviceType.StripeS700DevKit`              | `10`  |
| `DeviceType.TapToPay` (was `AppleBuiltIn`) | `11`  |
| `DeviceType.StripeS710`                    | `12`  |
| `DeviceType.StripeS710DevKit`              | `13`  |

**Action Required**: Replace any usage of `DeviceType.AppleBuiltIn` with `DeviceType.TapToPay`.

```typescript
// Before
if (reader.deviceType === DeviceType.AppleBuiltIn) { ... }

// After
if (reader.deviceType === DeviceType.TapToPay) { ... }
```

### 7. Capacitor v8 API Changes

Capacitor v8 includes some internal changes to plugin APIs, but the public API of this plugin remains the same. No changes are required in your application code related to Capacitor v8.

### 8. Removal of rxjs — Observable API Replaced with Callbacks

`rxjs` has been removed as a dependency. All methods that previously returned an `Observable` now accept a callback and return `Promise<PluginListenerHandle>`. Call `handle.remove()` to stop listening instead of calling `subscription.unsubscribe()`.

Affected methods:

| Method                                  | Before                                                               | After                                                                            |
| --------------------------------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `discoverReaders`                       | `discoverReaders(options): Observable<Reader[]>`                     | `discoverReaders(options, callback): Promise<PluginListenerHandle>`              |
| `connectionStatus`                      | `connectionStatus(): Observable<ConnectionStatus>`                   | `connectionStatus(callback): Promise<PluginListenerHandle>`                      |
| `didRequestReaderInput`                 | `didRequestReaderInput(): Observable<ReaderInputOptions>`            | `didRequestReaderInput(callback): Promise<PluginListenerHandle>`                 |
| `didRequestReaderDisplayMessage`        | `didRequestReaderDisplayMessage(): Observable<ReaderDisplayMessage>` | `didRequestReaderDisplayMessage(callback): Promise<PluginListenerHandle>`        |
| `didReportAvailableUpdate`              | `didReportAvailableUpdate(): Observable<ReaderSoftwareUpdate>`       | `didReportAvailableUpdate(callback): Promise<PluginListenerHandle>`              |
| `didStartInstallingUpdate`              | `didStartInstallingUpdate(): Observable<ReaderSoftwareUpdate>`       | `didStartInstallingUpdate(callback): Promise<PluginListenerHandle>`              |
| `didReportReaderSoftwareUpdateProgress` | `didReportReaderSoftwareUpdateProgress(): Observable<number>`        | `didReportReaderSoftwareUpdateProgress(callback): Promise<PluginListenerHandle>` |
| `didFinishInstallingUpdate`             | `didFinishInstallingUpdate(): Observable<...>`                       | `didFinishInstallingUpdate(callback): Promise<PluginListenerHandle>`             |
| `didStartReaderReconnect`               | `didStartReaderReconnect(): Observable<void>`                        | `didStartReaderReconnect(callback): Promise<PluginListenerHandle>`               |
| `didSucceedReaderReconnect`             | `didSucceedReaderReconnect(): Observable<void>`                      | `didSucceedReaderReconnect(callback): Promise<PluginListenerHandle>`             |
| `didFailReaderReconnect`                | `didFailReaderReconnect(): Observable<void>`                         | `didFailReaderReconnect(callback): Promise<PluginListenerHandle>`                |

**Action Required**: Remove any `rxjs` imports and update all affected call sites.

```typescript
// Before
import { Observable } from 'rxjs'

const discoverSub = terminal
  .discoverReaders({
    simulated: false,
    discoveryMethod: DiscoveryMethod.BluetoothScan,
  })
  .subscribe((readers) => {
    console.log('readers', readers)
  })

const displaySub = terminal
  .didRequestReaderDisplayMessage()
  .subscribe((message) => {
    console.log('displayMessage', message)
  })

const inputSub = terminal.didRequestReaderInput().subscribe((options) => {
  console.log('inputOptions', options)
})

// Later...
discoverSub.unsubscribe()
displaySub.unsubscribe()
inputSub.unsubscribe()

// After
const discoverHandle = await terminal.discoverReaders(
  { simulated: false, discoveryMethod: DiscoveryMethod.BluetoothScan },
  (readers) => {
    console.log('readers', readers)
  },
)

const displayHandle = await terminal.didRequestReaderDisplayMessage(
  (message) => {
    console.log('displayMessage', message)
  },
)

const inputHandle = await terminal.didRequestReaderInput((options) => {
  console.log('inputOptions', options)
})

// Later...
discoverHandle.remove()
displayHandle.remove()
inputHandle.remove()
```

## Breaking Changes Summary

The public API of `capacitor-stripe-terminal` has several breaking changes in this release. See each section above for full details.

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
   - Under "Deployment Info", set "Minimum Deployment" to iOS 15.0

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

This upgrade primarily updates the underlying SDKs while maintaining most API compatibility. The main actions required are:

- Update your iOS deployment target to 15.0
- Handle the new `Reconnecting` connection status if you monitor connection state
- On Android: replace `DiscoveryMethod.Embedded` and `DiscoveryMethod.Handoff` with `DiscoveryMethod.AppsOnDevices`; replace `connectHandoffReader` calls with `connectAppsOnDevicesReader`
- On Android: update any `DeviceType.VerifoneP400` handling to also handle `DeviceType.Unknown`
- Rename `DiscoveryMethod.LocalMobile` → `DiscoveryMethod.TapToPay`, `LocalMobileConnectionConfiguration` → `TapToPayConnectionConfiguration`, `connectLocalMobileReader()` → `connectTapToPayReader()`, and the `localMobileReaderDidAcceptTermsOfService` event → `tapToPayReaderDidAcceptTermsOfService`
- Rename `DeviceType.AppleBuiltIn` → `DeviceType.TapToPay` (now covers both iOS and Android Tap to Pay readers)
- **Remove `rxjs` from your dependencies** and update all Observable-based call sites to use the new callback + `PluginListenerHandle` pattern (call `handle.remove()` instead of `subscription.unsubscribe()`)
