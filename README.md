# miIO Device Library

This is a small library for controlling Mi Home devices that implement the miIO
protocol, such as the Mi Air Purifier, Mi Robot Vacuum and Mi Smart Socket.

## Installation

```
npm install --save miio
```

## Usage

```javascript
const miio = require('miio');
```

Create a handle to the device:

```javascript
// Create a device - with a model to get a more specific device instance
const device = miio.createDevice({
	token: 'token-as-hex', // Token of device
	model: 'zhimi.airpurifier.m1',
	address: '192.168.100.8'
});

// Create a new generic device by skipping the model
const device = miio.createDevice({
	token: 'token-as-hex', // Token of device
	address: '192.168.100.8'
});
```

Call methods to interact with the device:

```javascript
// Call any method via call
device.call('set_mode', [ 'silent' ])
	.then(console.log)
	.catch(console.error);

// Fetch some properties
device.getProperties([ 'power', 'mode', 'aqi', 'temp_dec', 'humidity' ])
	.then(console.log)
	.catch(console.error);

// Or if a model has been specified use some of its methods
device.setPower(false)
	.then(on => console.log('Power is now', on));
```

Monitor and transform properties:
```javascript
// Define a property that should be monitored
device.defineProperty('mode');

// Define that a certain property should be run through a custom conversion
device.defineProperty('temp_dec', v => v / 10.0);

// Listen for changes to properties
device.on('propertyChanged', e => console.log(e.property, e.oldValue, e.value));

// Activate automatic property monitoring (activated by default for most devices)
device.monitor();

// Stop automatic property monitoring
device.stopMonitoring();

// Fetch the last value of a monitored property
const value = device.property('temp_dec');
```

If you are done with the device call `destroy` to stop all network traffic:

```javascript
device.destroy();
```

## Tokens

A few Miio devices send back their token during a handshake and can be used
without figuring out the token. Most devices hide their token, such as
Yeelights and the Mi Robot Vacuum.

[Finding tokens](tokens.md) contains more information about how to get the
token of those devices.

## Discovering devices

Devices use mDNS for discovery, but this library does not contain a mDNS
implementation. You can choose a mDNS-implementation suitable for your
needs. Devices announce themselves via `_miio._udp` and should work for most
devices, in certain cases you might need to restart your device to make it
announce itself.

Here is an example on how to find devices with `tinkerhub-mdns`;

```javascript
const browser = require('tinkerhub-mdns')
	.browser({
		type: 'miio',
		protocol: 'udp'
	});

browser.on('available', reg => {
	// Use infoFromHostname to figure out model and id of device
	const info = miio.infoFromHostname(reg.name);
	if(! info) return;

	// Set the address and port from the found registration
	info.address = reg.addresses[0];
	info.port = reg.port;

	const device = miio.createDevice(info);

	// Do something with the device here
});

browser.on('unavailable', reg => {
	const device = ...;
	device.destroy();
});
```

## Devices types

The intent of this library is to encompass all miIO-compatible devices and to
provide and easy to use API for them. To do this types are mapped from their
model name into specific device types that provide a friendly API to the device.

Currently implemented devices are:

* Air Purifier
* Smart Socket Plug
* Mi Robot Vacuum

See [documentation for devices](devices.md) for information about the API for each
type.

## Advanced: Device management

Get information and update the wireless settings of devices via the management
API.

Discover the token of a device:
```
device.discover()
	.then(info => console.log(info.token));
```

Get internal information about the device:
```
device.management.info()
	.then(console.log);
```

Update the wireless settings:
```
device.management.updateWireless({
	ssid: 'SSID of network',
	passwd: 'Password of network'
}).then(console.log);
```

Warning: The device will either connect to the new network or it will end up
creating its own network if the settings are invalid.

## Protocol documentation

This library is based on the documentation provided by OpenMiHome. See https://github.com/OpenMiHome/mihome-binary-protocol for details.
