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

### Capacitor

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

### 4. `VerifoneP400` Device Type Removed

The `DeviceType.VerifoneP400` enum value has been removed from this plugin. The Verifone P400 countertop reader is no longer supported by the Stripe Terminal SDK v5.

**Action Required**: Remove any references to `DeviceType.VerifoneP400` in your code.

```typescript
// Before
if (reader.deviceType === DeviceType.VerifoneP400) {
  // handle Verifone P400
}

// After — remove this code path entirely; the Verifone P400 is no longer supported
```

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

### 9. `processPayment` Renamed to `confirmPaymentIntent`

The `processPayment` method has been renamed to `confirmPaymentIntent` to match the underlying native Stripe Terminal SDK function name.

| Before             | After                    |
| ------------------ | ------------------------ |
| `processPayment()` | `confirmPaymentIntent()` |

**Action Required**: Replace all calls to `processPayment()` with `confirmPaymentIntent()`.

```typescript
// Before
await terminal.processPayment()

// After
await terminal.confirmPaymentIntent()
```

### 10. `confirmPaymentIntent` Error Shape — Structured Decline Data

`confirmPaymentIntent` can now surface `decline_code` and `payment_intent` on the thrown `StripeTerminalError` when a card is declined.

Previously, catching a `confirmPaymentIntent` failure and reading `error.decline_code` or `error.payment_intent` always returned `undefined` because the native layers did not attach structured data to their rejections. This is now fixed on both platforms.

**Error shape (TypeScript):**

```typescript
try {
  await terminal.confirmPaymentIntent()
} catch (error) {
  if (error instanceof StripeTerminalError) {
    // Always present on decline — the human-readable reason
    console.log(error.message)

    // Now populated on card declines (was always undefined before)
    console.log(error.decline_code) // e.g. "insufficient_funds"
    console.log(error.payment_intent) // the updated PaymentIntent object (status: requires_payment_method)
  }
}
```

**Action Required**: If your code previously checked `error.decline_code` or `error.payment_intent` after a `confirmPaymentIntent` failure, those fields are now populated. No code changes are required to benefit from this, but you may want to add handling for them.

**Note on error guard change**: The internal `StripeTerminalError` construction was also made more robust. Previously, if the native side returned an error without a data payload, the raw error was re-thrown unmodified (not as a `StripeTerminalError`). Now, any error with a message is always wrapped in `StripeTerminalError`, with `decline_code` and `payment_intent` populated only when the native side provides them.

### 11. `Charge.status` Is Now a `ChargeStatus` Enum (was a string on Android / integer on iOS)

Previously, the `status` field of each `Charge` object in `PaymentIntent.charges` was inconsistent between platforms:

- **iOS** returned a raw integer (the `SCPChargeStatus` enum `rawValue`, e.g. `0`)
- **Android** returned a raw string (e.g. `"succeeded"`)
- **Web** passed through the Stripe SDK string directly (e.g. `"succeeded"`)

All three platforms now return the same `ChargeStatus` enum integer.

```typescript
export enum ChargeStatus {
  Succeeded = 0,
  Pending = 1,
  Failed = 2,
}
```

**Action Required**: Update any code that reads `charge.status`.

```typescript
// Before — unreliable, platform-dependent
if (charge.status === 'succeeded') { ... }   // only worked on Android/Web
if (charge.status === 0) { ... }             // only worked on iOS

// After — consistent across all platforms
import { ChargeStatus } from 'capacitor-stripe-terminal'

if (charge.status === ChargeStatus.Succeeded) { ... }
```

### 12. `PaymentIntent.charges` Now Uses Plugin-Local `Charge` Interface (was `Stripe.Charge`)

The type of `PaymentIntent.charges` has changed from `Stripe.Charge[]` (the Stripe Node.js SDK type) to `Charge[]`, a new interface defined in this plugin. This ensures the returned shape is consistent across iOS, Android, and Web and avoids pulling in a server-side SDK type for a client-side value.

The `Charge` interface includes all fields that were already being serialized by the native platforms:

```typescript
export interface Charge {
  stripeId: string
  amount: number
  currency: string
  status: ChargeStatus // enum — see breaking change #11
  metadata: { [key: string]: string }
  stripeDescription: string | null
  statementDescriptorSuffix: string | null
  calculatedStatementDescriptor: string | null
  authorizationCode: string | null
  amountRefunded: number
  created: number | null // Unix timestamp (seconds)
  captured: boolean
  paid: boolean
  refunded: boolean
  customer: string | null
  paymentIntentId: string | null
  receiptEmail: string | null
  receiptNumber: string | null
  receiptUrl: string | null
  livemode: boolean
}
```

**Action Required**: If your code imports `Stripe.Charge` from the `stripe` package to type charge objects returned by this plugin, switch to importing `Charge` from `capacitor-stripe-terminal` instead.

```typescript
// Before
import { Stripe } from 'stripe'
const charge: Stripe.Charge = paymentIntent.charges[0]

// After
import { Charge } from 'capacitor-stripe-terminal'
const charge: Charge = paymentIntent.charges[0]
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
- Remove any usage of `DeviceType.VerifoneP400` — the Verifone P400 reader is no longer supported
- Rename `DiscoveryMethod.LocalMobile` → `DiscoveryMethod.TapToPay`, `LocalMobileConnectionConfiguration` → `TapToPayConnectionConfiguration`, `connectLocalMobileReader()` → `connectTapToPayReader()`, and the `localMobileReaderDidAcceptTermsOfService` event → `tapToPayReaderDidAcceptTermsOfService`
- Rename `DeviceType.AppleBuiltIn` → `DeviceType.TapToPay` (now covers both iOS and Android Tap to Pay readers)
- **Remove `rxjs` from your dependencies** and update all Observable-based call sites to use the new callback + `PluginListenerHandle` pattern (call `handle.remove()` instead of `subscription.unsubscribe()`)
- Rename `processPayment()` → `confirmPaymentIntent()` (matches the native Stripe Terminal SDK function name)
- `confirmPaymentIntent` errors are now always thrown as `StripeTerminalError`; `decline_code` and `payment_intent` fields are now populated on card declines (no action required, but you may now read these fields in your catch handler)
