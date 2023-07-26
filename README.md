[![pub package](https://img.shields.io/pub/v/flutter_blue_plus.svg)](https://pub.dartlang.org/packages/flutter_blue_plus)

<br>
<p align="center">
<img alt="FlutterBlue" src="https://github.com/boskokg/flutter_blue_plus/blob/master/site/flutterblue.png?raw=true" />
</p>
<br><br>

**Note: this plugin is continuous work from FlutterBlue since maintenance stopped.**

## Contents

- [Introduction](#introduction)
- [Usage](#usage)
- [Getting Started](#getting-started)
- [Reference](#reference)
- [Debugging](#debugging)
- [Common Problems](#common-problems)

## Introduction

FlutterBluePlus is a bluetooth plugin for [Flutter](https://flutter.dev), a new app SDK to help developers build modern multi-platform apps.

## Cross-Platform Bluetooth LE

FlutterBluePlus aims to offer the most from all supported platforms: iOS, macOS, Android.

The code is written to be simple, robust, and incredibly easy to understand.

## No Dependencies

FlutterBluePlus has zero dependencies besides Flutter, Android, and iOS themselves.

This makes FlutterBluePlus very stable.

## Usage

### Error Handling :fire:

Flutter Blue Plus takes error handling very seriously. 

Every error returned by the native platform is checked and thrown as an exception where appropriate. See [Reference](#reference) for a list of throwable functions.

**Streams:** At the time of writing, streams returned by Flutter Blue Plus never emit any errors and never close. There's no need to handle `onError` or `onDone` for  `stream.listen(...)`.

---

### Enable Bluetooth

```dart
// check availability
if (await FlutterBluePlus.isAvailable == false) {
    print("Bluetooth not supported by this device");
    return;
}

// turn on bluetooth ourself if we can
if (Platform.isAndroid) {
    await FlutterBluePlus.turnOn();
}

// wait bluetooth to be on
await FlutterBluePlus.adapterState.where((s) => s == BluetoothAdapterState.on).first;
```

### Scan for devices

If your device is not found, see [Common Problems](#common-problems).

```dart
// Setup Listener for scan results
// device not found? see "Common Problems" in the README
var subscription = FlutterBluePlus.scanResults.listen((results) {
    for (ScanResult r in results) {
        print('${r.device.localName} found! rssi: ${r.rssi}');
    }
});

// Start scanning
FlutterBluePlus.startScan(timeout: Duration(seconds: 4));

// Stop scanning
await FlutterBluePlus.stopScan();
```

### Connect to a device

```dart
// Connect to the device
await device.connect();

// Disconnect from device
device.disconnect();
```

### Discover services

```dart
List<BluetoothService> services = await device.discoverServices();
services.forEach((service) {
    // do something with service
});
```

### Read and write characteristics

```dart
// Reads all characteristics
var characteristics = service.characteristics;
for(BluetoothCharacteristic c in characteristics) {
    List<int> value = await c.read();
    print(value);
}

// Writes to a characteristic
await c.write([0x12, 0x34])
```

### Read and write descriptors

```dart
// Reads all descriptors
var descriptors = characteristic.descriptors;
for(BluetoothDescriptor d in descriptors) {
    List<int> value = await d.read();
    print(value);
}

// Writes to a descriptor
await d.write([0x12, 0x34])
```

### Set notifications and listen to changes

If onValueReceived is never called, see [Common Problems](#common-problems) in the README.

```dart
// Setup Listener for characteristic reads
// If this is never called, see "Common Problems" in the README
characteristic.onValueReceived.listen((value) {
    // do something with new value
});

// enable notifications
await characteristic.setNotifyValue(true);
```

### Read the MTU and request larger size

```dart
final mtu = await device.mtu.first;
await device.requestMtu(512);
```

Note that iOS will not allow requests of MTU size, and will always try to negotiate the highest possible MTU (iOS supports up to MTU size 185)

## Getting Started

### Change the minSdkVersion for Android

flutter_blue_plus is compatible only from version 19 of Android SDK so you should change this in **android/app/build.gradle**:

```dart
Android {
  defaultConfig {
     minSdkVersion: 19
```

### Add permissions for Android (No Location)

In the **android/app/src/main/AndroidManifest.xml** add:

```xml
<!-- Tell Google Play Store that your app uses Bluetooth LE
     Set android:required="true" if bluetooth is necessary -->
<uses-feature android:name="android.hardware.bluetooth_le" android:required="false" />

<!-- New Bluetooth permissions in Android 12
https://developer.android.com/about/versions/12/features/bluetooth-permissions -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />

<!-- legacy for Android 11 or lower -->
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" android:maxSdkVersion="30"/>


<!-- legacy for Android 9 or lower -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" android:maxSdkVersion="28" />
```

### Add permissions for Android (With Fine Location)

If you want to use Bluetooth to determine location.

In the **android/app/src/main/AndroidManifest.xml** add:

```xml
<!-- Tell Google Play Store that your app uses Bluetooth LE
     Set android:required="true" if bluetooth is necessary -->
<uses-feature android:name="android.hardware.bluetooth_le" android:required="false" />

<!-- New Bluetooth permissions in Android 12
https://developer.android.com/about/versions/12/features/bluetooth-permissions -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<!-- legacy for Android 11 or lower -->
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />


<!-- legacy for Android 9 or lower -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" android:maxSdkVersion="28" />
```

```dart
// Start scanning
flutterBlue.startScan(timeout: Duration(seconds: 4), androidUsesFineLocation: true);
```

### Android Proguard

Add the following line in your `project/android/app/proguard-rules.pro` file:

```
-keep class com.boskokg.flutter_blue_plus.* { *; }
```

to avoid seeing the following kind errors in your `release` builds:

```
PlatformException(startScan, Field androidScanMode_ for m0.e0 not found. Known fields are
 [private int m0.e0.q, private b3.b0$i m0.e0.r, private boolean m0.e0.s, private static final m0.e0 m0.e0.t,
 private static volatile b3.a1 m0.e0.u], java.lang.RuntimeException: Field androidScanMode_ for m0.e0 not found
```

### Add permissions for iOS

In the **ios/Runner/Info.plist** let’s add:

```dart
	<dict>
	    <key>NSBluetoothAlwaysUsageDescription</key>
	    <string>Need BLE permission</string>
	    <key>NSBluetoothPeripheralUsageDescription</key>
	    <string>Need BLE permission</string>
	    <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
	    <string>Need Location permission</string>
	    <key>NSLocationAlwaysUsageDescription</key>
	    <string>Need Location permission</string>
	    <key>NSLocationWhenInUseUsageDescription</key>
	    <string>Need Location permission</string>
```

For location permissions on iOS see more at: [https://developer.apple.com/documentation/corelocation/requesting_authorization_for_location_services](https://developer.apple.com/documentation/corelocation/requesting_authorization_for_location_services)

## Reference

### FlutterBlue API

|                        |      Android       |        iOS         | Throws | Description                                                |
| :--------------------- | :----------------: | :----------------: | :----: | :----------------------------------------------------------|
| adapterState           | :white_check_mark: | :white_check_mark: |        | Stream of state changes for the bluetooth adapter          |
| isAvailable            | :white_check_mark: | :white_check_mark: |        | Checks whether the device supports Bluetooth               |
| isOn                   | :white_check_mark: | :white_check_mark: |        | Checks if Bluetooth adapter is turned on                   |
| turnOn                 | :white_check_mark: |                    | :fire: | Turns on the bluetooth adapter                             |
| turnOff                | :white_check_mark: |                    | :fire: | Turns off the bluetooth adapter                            |
| scan                   | :white_check_mark: | :white_check_mark: | :fire: | Starts a scan for Ble devices and returns a stream         |
| startScan              | :white_check_mark: | :white_check_mark: | :fire: | Starts a scan for Ble devices with no return value         |
| stopScan               | :white_check_mark: | :white_check_mark: | :fire: | Stop an existing scan for Ble devices                      |
| scanResults            | :white_check_mark: | :white_check_mark: |        | Stream of live scan results                                |
| isScanning             | :white_check_mark: | :white_check_mark: |        | Stream of current scanning state                           |
| isScanningNow          | :white_check_mark: | :white_check_mark: |        | Is a scan currently running?                               |
| connectedSystemDevices | :white_check_mark: | :white_check_mark: |        | List of already connected devices, including by other apps |
| setLogLevel            | :white_check_mark: | :white_check_mark: |        | Configure plugin log level                                 |

### BluetoothDevice API

|                           |      Android       |        iOS         | Throws | Description                                                |
| :------------------------ | :----------------: | :----------------: | :----: | :----------------------------------------------------------|
| localName                 | :white_check_mark: | :white_check_mark: |        | The cached localName of the device                         |
| connect                   | :white_check_mark: | :white_check_mark: | :fire: | Establishes a connection to the device                     |
| disconnect                | :white_check_mark: | :white_check_mark: | :fire: | Cancels an active or pending connection to the device      |
| discoverServices          | :white_check_mark: | :white_check_mark: | :fire: | Discover services                                          |
| isDiscoveryingServices    | :white_check_mark: | :white_check_mark: |        | Stream of whether service discovery is in progress         |
| servicesList              | :white_check_mark: | :white_check_mark: |        | The list of services that were discovered                  |
| servicesStream            | :white_check_mark: | :white_check_mark: |        | Stream of services changes                                 |
| connectionState           | :white_check_mark: | :white_check_mark: |        | Stream of connection changes for the Bluetooth Device      |
| mtu                       | :white_check_mark: | :white_check_mark: | :fire: | Stream of mtu size changes                                 |
| readRssi                  | :white_check_mark: | :white_check_mark: | :fire: | Read RSSI from a connected device                          |
| requestMtu                | :white_check_mark: |                    | :fire: | Request to change the MTU for the device                   |
| requestConnectionPriority | :white_check_mark: |                    | :fire: | Request to update a high priority, low latency connection  |
| pair                      | :white_check_mark: |                    | :fire: | Calls createBond on a device                               |
| removeBond                | :white_check_mark: |                    | :fire: | Remove Bluetooth Bond of device                            |
| setPreferredPhy           | :white_check_mark: |                    |        | Set preferred RX and TX phy for connection and phy options |
| clearGattCache            | :white_check_mark: |                    | :fire: | Clear android cache of service discovery results           |

### BluetoothCharacteristic API

|                 |      Android       |        iOS         | Throws | Description                                                    |
| :-------------  | :----------------: | :----------------: | :----: | :--------------------------------------------------------------|
| uuid            | :white_check_mark: | :white_check_mark: |        | The uuid of characeristic                                      |
| read            | :white_check_mark: | :white_check_mark: | :fire: | Retrieves the value of the characteristic                      |
| write           | :white_check_mark: | :white_check_mark: | :fire: | Writes the value of the characteristic                         |
| setNotifyValue  | :white_check_mark: | :white_check_mark: | :fire: | Sets notifications or indications on the characteristic        |
| isNotifying     | :white_check_mark: | :white_check_mark: |        | Are notifications or indications currently enabled             |
| onValueReceived | :white_check_mark: | :white_check_mark: |        | Stream of characteristic value updates received from the device|
| lastValue       | :white_check_mark: | :white_check_mark: |        | The most recent value of the characteristic                    |
| lastValueStream | :white_check_mark: | :white_check_mark: |        | Stream of lastValue + onValueReceived                          |

### BluetoothDescriptor API

|                   |      Android       |        iOS         | Throws | Description                                    |
| :----             | :----------------: | :----------------: | :----: | :----------------------------------------------|
| uuid              | :white_check_mark: | :white_check_mark: |        | The uuid of descriptor                         |
| read              | :white_check_mark: | :white_check_mark: | :fire: | Retrieves the value of the descriptor          |
| write             | :white_check_mark: | :white_check_mark: | :fire: | Writes the value of the descriptor             |
| onValueReceived   | :white_check_mark: | :white_check_mark: |        | Stream of descriptor value reads & writes      |
| lastValue         | :white_check_mark: | :white_check_mark: |        | The most recent value of the descriptor        |
| lastValueStream   | :white_check_mark: | :white_check_mark: |        | Stream of lastValue + onValueReceived          |

## Debugging

The easiest way to debug issues in FlutterBluePlus is to make your own local copy.

```
cd /user/downloads
git clone https://github.com/boskokg/flutter_blue_plus.git
```

then in `pubspec.yaml` add the repo by path:

```
  flutter_blue_plus:
    path: /user/downloads/flutter_blue_plus
```

Now you can edit the FlutterBluePlus code yourself.

## Common Problems

Many common problems are easily solved.

---

### Scanning does not find my device

**1. your device uses bluetooth classic, not BLE.**

Headphones, speakers, keyboards, mice, gamepads, & printers all use Bluetooth Classic. 

These devices may be found in System Settings, but they cannot be connected to by FlutterBluePlus. FlutterBluePlus only supports Bluetooth Low Energy.

**2. your device stopped advertising.**

- you might need to reboot your device
- you might need put your device in "discovery mode"
- your phone may have already connected automatically
- another app may have already connected to your device

Try looking through already connected devices:

```dart
// search already connected devices, including devices
// connected to by other apps
List<BluetoothDevice> system = await FlutterBluePlus.connectedSystemDevices;
for (var d in system) {
    print('${r.device.localName} already connected to! ${r.device.remoteId}');
    if (d.localName == "myBleDevice") {
         await r.connect(); // must connect our app
    }
}
```

**3. your scan filters are wrong.**

- try removing all scan filters
- for `withServices` to work, your device must actively advertise the serviceUUIDs it supports


**4. try a ble scanner app**

Search the App Store for a BLE scanner apps. 

You should check if they can discover your device.

---

### onValueReceived is never called

**1. you are not subscribed**

Your device will only send values after you call `await characteristic.setNotifyValue(true)`

**2. you are calling write**

`onValueReceived` is only called for reads & notifies.

You can do a single read with `await characteristic.read(...)`

**3. your device has nothing to send**

If you are using `setNotifyValue`, your device chooses when to send data.

Try interacting with your device to get it to send new data.

**4. your device has bugs**

Try rebooting your ble device. 

Some ble devices have buggy software and stop sending data.









