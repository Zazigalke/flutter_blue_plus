[![pub package](https://img.shields.io/pub/v/flutter_blue_plus.svg)](https://pub.dartlang.org/packages/flutter_blue_plus)

<br>
<p align="center">
<img alt="FlutterBlue" src="https://github.com/boskokg/flutter_blue_plus/blob/master/site/flutterblue.png?raw=true" />
</p>
<br><br>

**Note: this plugin is continuous work from FlutterBlue since maintenance stopped.**

## Foreward

For simple BLE apps, you should also consider QuickBlue (https://pub.dev/packages/quick_blue). Fewer features, smaller codebase.

## Introduction

FlutterBluePlus is a bluetooth plugin for [Flutter](https://flutter.dev), a new app SDK to help developers build modern multi-platform apps.

## Cross-Platform Bluetooth LE

FlutterBluePlus aims to offer the most from all supported platforms: iOS, macOS, Android. (Feel free to contribute Windows support!)

Using the FlutterBluePlus instance, you can scan for and connect to nearby devices ([BluetoothDevice](#bluetoothdevice-api)).
Once connected to a device, the BluetoothDevice object can discover services ([BluetoothService](lib/src/bluetooth_service.dart)), characteristics ([BluetoothCharacteristic](lib/src/bluetooth_characteristic.dart)), and descriptors ([BluetoothDescriptor](lib/src/bluetooth_descriptor.dart)).
The BluetoothDevice object is then used to directly interact with characteristics and descriptors.

## Troubleshooting

The easiest way to debug issues in FlutterBluePlus is to first make local copy.

```
cd /user/downloads
git clone https://github.com/boskokg/flutter_blue_plus.git
```

then in `pubspec.yaml` add the repo by path:

```
  flutter_blue_plus:
    path: /user/downloads/flutter_blue_plus
```

Now you can edit FlutterBluePlus code and debug issues yourself. 

### When I scan using a service UUID filter, it doesn't find any devices.

Make sure the device is advertising which service UUID's it supports. This is found in the advertisement
packet as **UUID 16 bit complete list** or **UUID 128 bit complete list**.

## Usage

### Scan for devices

```dart
// Start scanning
FlutterBluePlus.startScan(timeout: Duration(seconds: 4));

// Listen to scan results
var subscription = FlutterBluePlus.scanResults.listen((results) {
    // do something with scan results
    for (ScanResult r in results) {
        print('${r.device.localName} found! rssi: ${r.rssi}');
    }
});

// Stop scanning
FlutterBluePlus.stopScan();
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

```dart
await characteristic.setNotifyValue(true);
characteristic.onValueReceived.listen((value) {
    // do something with new value
});
```

### Read the MTU and request larger size

```dart
final mtu = await device.mtu.first;
await device.requestMtu(512);
```

Note that iOS will not allow requests of MTU size, and will always try to negotiate the highest possible MTU (iOS supports up to MTU size 185)

## Error Handling

**All functions in FlutterBluePlus will throw exceptions when they encounter errors.** 

 **You MUST handle exceptions for bluetooth communication:**
- `BluetoothDevice.connect()`
- `BluetoothDevice.disconnect()`
- `BluetoothDevice.discoverServices()`
- `BluetoothDevice.requestMtu()`
- `BluetoothDevice.readRssi()`
- `BluetoothDevice.pair()`
- `BluetoothCharacteristic.read()`
- `BluetoothCharacteristic.write()`
- `BluetoothCharacteristic.setNotifyValue()`
- `BluetoothDescriptor.read()`
- `BluetoothDescriptor.read()`

 **These functions also THROW:**
- `FlutterBluePlus.turnOn()`
- `FlutterBluePlus.turnOff()`
- `FlutterBluePlus.startScan()`
- `FlutterBluePlus.stopScan()`
- `FlutterBluePlus.scan()`
- `BluetoothDevice.requestConnectionPriority`
- `BluetoothDevice.mtu`
- `BluetoothDevice.removeBond()`
- `BluetoothDevice.clearGattCache()`

 **At the time of writing, these function DO NOT throw exceptions**
- `FlutterBluePlus.isAvailable`
- `FlutterBluePlus.isOn`
- `FlutterBluePlus.adapterState`
- `FlutterBluePlus.isScanning`
- `FlutterBluePlus.isScanningNow`
- `FlutterBluePlus.scanResults`
- `FlutterBluePlus.setLogLevel()`
- `BluetoothDevice.localName`
- `BluetoothDevice.connectionState`
- `BluetoothDevice.setPreferredPhy()`
- `BluetoothCharacteristic.isNotifying`
- `BluetoothCharacteristic.lastValue, uuid, & other simple accessors`
- `BluetoothDescriptor.lastValue, uuid, & other simple accessors`

**Example Code:**

```dart
try {
    await characteristic.read();
} catch (e) {
    print(userFriendlyError(e))
}

String userFriendlyError(dynamic e) {
  if (e is FlutterBluePlusException) {return e.errorString;}
  if (e is PlatformException) {return e.message;}
  return e.toString();
}
```

### Bluetooth Streams

The Bluetooth receive streams **do not** return errors or close. These include:
- `BluetoothCharacteristic.onValueReceived`
- `BluetoothCharacteristic.lastValueStream`
- `BluetoothDescriptor.onValueReceived`
- `BluetoothDescriptor.lastValueStream`

This is a limitation of the bluetooth protocol.

These streams stio receiving values after the connection is lost so make sure to listen to `BluetoothDevice.connectionState` and respond to it accordingly.

### Other Streams

These streams **never** encounter any errors of any kind.

- `FlutterBluePlus.isScanning`
- `FlutterBluePlus.scanResults`
- `FlutterBluePlus.adapterState`
- `BluetoothDevice.connectionState`
- `BluetoothDevice.isDiscoveringServices`

These streams **could** throw an exception when first called if the remoteId is no longer valid.
- `BluetoothDevice.services`
- `BluetoothDevices.mtu`

## Getting Started

### Change the minSdkVersion for Android

flutter_blue_plus is compatible only from version 19 of Android SDK so you should change this in **android/app/build.gradle**:

```dart
Android {
  defaultConfig {
     minSdkVersion: 19
```

### Add permissions for Bluetooth

We need to add the permission to use Bluetooth and access location:

#### **Android**

In the **android/app/src/main/AndroidManifest.xml** let’s add:

```xml
	  <uses-permission android:name="android.permission.BLUETOOTH_SCAN" android:usesPermissionFlags="neverForLocation" />
	  <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
 <application
```

By default the package does not request the ACCESS_FINE_LOCATION permission on Android 12+. In case you need to get the physical
location of the device via Bluetooth, you need to add the permission to the **android/app/src/main/AndroidManifest.xml**:

```xml
	  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
 <application
```

and set the androidUsesFineLocation flag to true when scanning:

```dart
// Start scanning
flutterBlue.startScan(timeout: Duration(seconds: 4), androidUsesFineLocation: true);

// Stop scanning
flutterBlue.stopScan();
```

#### **IOS**

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

|             |      Android       |        iOS         | Description                                        |
| :---------- | :----------------: | :----------------: | :------------------------------------------------- |
| scan        | :white_check_mark: | :white_check_mark: | Starts a scan for Bluetooth Low Energy devices.    |
| adapterState| :white_check_mark: | :white_check_mark: | Stream of state changes for the Bluetooth Adapter. |
| isAvailable | :white_check_mark: | :white_check_mark: | Checks whether the device supports Bluetooth.      |
| isOn        | :white_check_mark: | :white_check_mark: | Checks if Bluetooth functionality is turned on.    |

### BluetoothDevice API

|                           |      Android       |        iOS         | Description                                                                                                                                                                          |
| :------------------------ | :----------------: | :----------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| connect                   | :white_check_mark: | :white_check_mark: | Establishes a connection to the device.                                                                                                                                              |
| disconnect                | :white_check_mark: | :white_check_mark: | Cancels an active or pending connection to the device.                                                                                                                               |
| discoverServices          | :white_check_mark: | :white_check_mark: | Discovers services offered by the remote device as well as their characteristics and descriptors.                                                                                    |
| services                  | :white_check_mark: | :white_check_mark: | Gets a list of services. Requires that discoverServices() has completed.                                                                                                             |
| connectionState           | :white_check_mark: | :white_check_mark: | Stream of connection changes for the Bluetooth Device.                                                                                                                                    |
| mtu                       | :white_check_mark: | :white_check_mark: | Stream of mtu size changes.                                                                                                                                                          |
| requestMtu                | :white_check_mark: |                    | Request to change the MTU for the device.                                                                                                                                            |
| readRssi                  | :white_check_mark: | :white_check_mark: | Read RSSI from a connected device.                                                                                                                                                   |
| requestConnectionPriority | :white_check_mark: |                    | Request to update a high priority, low latency connection. An application should only request high priority connection parameters to transfer large amounts of data over LE quickly. |
| removeBond                | :white_check_mark: |                    | Remove Bluetooth Bond of device                                                                                                                                                      |
| setPreferredPhy           | :white_check_mark: |                    | Set preferred RX and TX phy for connection and phy options                                                                                                                           |

### BluetoothCharacteristic API

|                 |      Android       |        iOS         | Description                                              |
| :-------------  | :----------------: | :----------------: | :------------------------------------------------------- |
| read            | :white_check_mark: | :white_check_mark: | Retrieves the value of the characteristic.               |
| write           | :white_check_mark: | :white_check_mark: | Writes the value of the characteristic.                  |
| setNotifyValue  | :white_check_mark: | :white_check_mark: | Sets notifications or indications on the characteristic. |
| onValueReceived | :white_check_mark: | :white_check_mark: | Stream of characteristic value changes                   |
| lastValueStream | :white_check_mark: | :white_check_mark: | Stream of lastValue + characteristic value changes       |

### BluetoothDescriptor API

|                 |      Android       |        iOS         | Description                                    |
| :----           | :----------------: | :----------------: | :-------------------------------------         |
| read            | :white_check_mark: | :white_check_mark: | Retrieves the value of the descriptor.         |
| write           | :white_check_mark: | :white_check_mark: | Writes the value of the descriptor.            |
| onValueReceived | :white_check_mark: | :white_check_mark: | Stream of descriptor value changes             |
| lastValueStream | :white_check_mark: | :white_check_mark: | Stream of lastValue + descriptor value changes |


