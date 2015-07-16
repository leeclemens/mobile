# `iosrtc` API

The top level `iosrtc` module is a JavaScript Object containing all the WebRTC classes and functions.

The `iosrtc` module is exposed within the `window.cordova.plugins` namespace (Cordova plugins convention). Example:

```javascript
var pc = new cordova.plugins.iosrtc.RTCPeerConnection({
  iceServers: []
});

cordova.plugins.iosrtc.getUserMedia(
  // constraints
  { audio: true, video: true },
  // success callback
  function (stream) {
    console.log('got local MediaStream: ', stream);

    pc.addStream(stream);
  },
  // failure callback
  function (error) {
    console.error('getUserMedia failed: ', error);
  }
);
```


### `iosrtc.getUserMedia()`

Implementation of the  `getUserMedia()` function as specified by the [W3C Media Capture and Streams draft](http://w3c.github.io/mediacapture-main/#local-content).

The function allows both the old/deprecated callbacks based syntax and the new one based on Promises (depending on the number and type of the given arguments).

*NOTE:* In iOS devices there is a single audio input (mic) and two video inputs (camera). If the given constraints include "video" the device chosen by default is the front camera. However the back camera can be chosen by passing an "optional" or "mandatory" constraint to the function:

```javascript
cordova.plugins.iosrtc.getUserMedia({
  audio: true,
  video: {
    optional: [
      { sourceId: 'com.apple.avfoundation.avcapturedevice.built-in_video:1' }
    ]
  }
});
```

*NOTE:* The API to select a specific device is outdated, but it matches the one currently implemented by Chrome browser.

*TODO:*

* Rich constraints.


### `iosrtc.getMediaDevices()`

Implementation of the  `enumerateDevices()` function as specified in the [W3C Media Capture and Streams draft](http://w3c.github.io/mediacapture-main/#enumerating-devices).

The function allows both the old/deprecated callbacks based syntax and the new one based on Promises.

The success callback is called with a list of `MediaDeviceInfo` objects as defined in the same spec. However such an object includes deprecated fields for backwards compatibility. The read-only fields in a `MediaDeviceInfo` object are:

* `deviceId` (String)
* `kind` (String)
* `label` (String)
* `groupId` (always an empty string)
* `id` (same as `deviceId`, deprecated)
* `facing` (always an empty string, deprecated)

*NOTE:* The `deviceId` or `id` field is the value to be used in the `sourceId` field of `getUserMedia()` above to choose a specific device.


### `iosrtc.RTCPeerConnection`

Exposes the `RTCPeerConnection` class as defined by the [W3C Real-time Communication Between Browsers draft](http://www.w3.org/TR/webrtc/#rtcpeerconnection-interface).

All the methods are implemented in both fashions: the deprecated callbacks based syntax and the new one based on Promises.

*TODO:*

* `updateIce()` method.
* `getStats()` method.
* Can not use `id` value greater than 1023 in the config object for `createDataChannel()` (see [issue #4618](https://code.google.com/p/webrtc/issues/detail?id=4618)).


###  `iosrtc.RTCSessionDescription`

Exposes the `RTCSessionDescription` class as defined by the [spec](http://www.w3.org/TR/webrtc/#idl-def-RTCSessionDescription).


### `iosrtc.RTCIceCandidate`

Exposes the `RTCIceCandidate` class as defined by the [spec](http://www.w3.org/TR/webrtc/#idl-def-RTCIceCandidate).


### `iosrtc.MediaStream`

Exposes the  `MediaStream` class as defined in the [spec](http://w3c.github.io/mediacapture-main/#mediastream).

*NOTES:*

* For internal reasons the `MediaStream` class points to the [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) class, so the `MediaStream` class constructor is not implemented (this class is exposed to make some WebRTC polyfill libraries happy).
* `stop()` method implemented for backwards compatibility (it calls `stop()` on all its `MediaStreamTracks`).

*TODO:*

* `clone()` method.


### `iosrtc.MediaStreamTrack`

Exposes the `MediaStreamTrack` class as defined by the [spec](http://w3c.github.io/mediacapture-main/#mediastreamtrack).

*NOTE:* The only reason to make this class public is to expose the deprecated `MediaStreamTrack.getSources()` class function, which is an "alias" to the `getMediaDevices()` function described above.

*TODO:*

* `muted` attribute (not exposed by the Objective-C wrapper of the Google WebRTC library).
* `onmute` and `onunmute` events.
* `clone()` methods.
* `getCapabilities()` method.
* `getConstraints()` method.
* `getSettings()` method.
* `applyConstraints()` method.
* `onoverconstrained` event.


### `iosrtc.refreshVideos()`

When calling this method, the height/width, opacity, visibility and z-index of all the HTML5 video elements rendering a `MediaStream` are recomputed and the iOS native `UIView` layer updated according.

Call this method when the position or size of a video element changes.


### `iosrtc.selectAudioOutput(output)`

Select the audio output device. Given `output` argument must be "earpiece" or "speaker".

*NOTE:* "speaker" output can only be set during a WebRTC session.


### `iosrtc.registerGlobals()`

By calling this method the JavaScript global namespace gets "polluted" with the following additions:

* `navigator.getUserMedia`
* `navigator.webkitGetUserMedia`
* `navigator.mediaDevices.getUserMedia`
* `navigator.mediaDevices.enumerateDevices`
* `window.RTCPeerConnection`
* `window.webkitRTCPeerConnection`
* `window.RTCSessionDescription`
* `window.RTCIceCandidate`
* `window.MediaStream`
* `window.webkitMediaStream`
* `window.MediaStreamTrack`

Useful to avoid iOS specified code in your HTML5 application.


### `iosrtc.debug`

The [debug](https://github.com/visionmedia/debug) module. Useful to enable verbose logging:

```javascript
cordova.plugins.iosrtc.debug.enable('iosrtc*');
```


### `iosrtc.rtcninjaPlugin`

A plugin interface for [rtcninja](https://github.com/eface2face/rtcninja.js/). 

Usage (assuming that [Cordova Device Plugin](http://plugins.cordova.io/#/package/org.apache.cordova.device) is installed):

```javascript
// Just for Cordova apps.
document.addEventListener('deviceready', function () {
  // Just for iOS devices.
  if (window.device.platform === 'iOS') {
    // Load rtcninja with cordova-plugin-iosrtc.
    rtcninja({
      plugin: cordova.plugins.iosrtc.rtcninjaPlugin
    });
  }

  console.log('WebRTC supported?: %s', rtcninja.hasWebRTC());
  // => WebRTC supported?: true

  rtcninja.RTCPeerConnection === cordova.plugins.iosrtc.RTCPeerConnection;
  // => true
});
```



## Others


### `RTCDataChannel`

The `RTCDataChannel` class (as defined in the [spec](http://www.w3.org/TR/webrtc/#idl-def-RTCDataChannel)) is not directly exposed by `iosrtc` via public API. Instead an instance of `RTCDataChannel` is returned by `createDataChannel()` and provided by the `ondatachannel` event.

The full DataChannel API is implemented (including binary messages).

*TODO:*

* `binaryType` just accepts `arraybuffer` (same as Chrome browser).