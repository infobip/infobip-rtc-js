### WebRTC SDK 1.x deprecation
Following the major release of our new RTC SDK 2.0, we are deprecating the SDK 1.x releases. The SDK 1.x will be out of
service on 31/10/2023. All new WebRTC customers must use the SDK 2.x, and customers still using SDK 1.x must migrate
to the newer release before the end of service date. To migrate from RTC SDK 1.x to 2.x, consult our
[migration guides](https://github.com/infobip/infobip-rtc-js/wiki/Migration-overview).
The deprecated [SDK 1.x Github repository](https://github.com/infobip/infobip-rtc-js-1.x-deprecated) can still be
consulted until the end of service date.

### Introduction

Infobip RTC is a JavaScript SDK which enables you to take advantage of Infobip platform,
giving you the ability to enrich your web applications with real-time communications in minimum time,
while you focus on your application's user experience and business logic.
We currently support WebRTC calls between two web or app users, phone calls between a web or app user and an actual 
phone, Viber calls, calls to the Infobip Conversations platform, as well as room calls - calls with multiple 
participants.

Here you will find an overview, and a quick guide on how to connect to Infobip platform.
There is also in-depth reference documentation available.

### Prerequisites

Infobip RTC SDK requires ES6. Also secure connection over HTTPS is required, except for localhost address.

### First-time setup

In order to use Infobip RTC, you need to have Web and In-app Calls enabled on your account and that's it!
You are ready to make Web and In-app calls. To learn how to enable them
see [the documentation](https://www.infobip.com/docs/voice-and-video/webrtc#set-up-web-and-in-app-calls).

### Getting SDK

There are a few ways in which you can get our SDK.
We publish it as an NPM package and as a standalone JS file hosted on a CDN.

If you want to add it as an NPM dependency, run the following:

```bash
npm install infobip-rtc --save
```

After which you would use it in your project like this:

```javascript
let InfobipRTC = require('infobip-rtc');
```

or as ES6 import:

```javascript
import {InfobipRTC} from "infobip-rtc";
```

You can include our distribution file in your JavaScript from our CDN:

```html
<script src="//rtc.cdn.infobip.com/2.0.0/infobip.rtc.js"></script>
```

The latest tag is also available:

```html
<script src="//rtc.cdn.infobip.com/latest/infobip.rtc.js"></script>
```

### Authentication

Since Infobip RTC is an SDK, it means you develop your own application, and you only use Infobip RTC as a dependency.
Your application has your own users, which we will call subscribers throughout this guide.
So, in order to use Infobip RTC, you need to register your subscribers on our platform.
The credentials your subscribers use to connect to your application are irrelevant to Infobip.
We only need the identity they will use to present themselves. When we have the subscriber's identity,
we can generate a token assigned to that specific subscriber.
With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

To generate these tokens for your subscribers, you need to call our
[`/webrtc/1/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method using proper parameters.
There you authenticate yourself against Infobip platform, so we can relate the subscriber's token to you.
Typically, generating a token occurs after your subscribers are authenticated inside your application.
You will receive the token in a response that you will use to instantiate
[`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client in your web application.

### Infobip RTC Client

After you have received a token via HTTP API,
you are ready to create an instance of the [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) 
client. You can do that using the [`createInfobipRtc`](https://github.com/infobip/infobip-rtc-js/wiki/Creating-InfobipRTC-Client)
global function.

```javascript
let token = obtainToken(); // here you call '/webrtc/1/token'
let options = {debug: true}
let infobipRTC = createInfobipRtc(token, options);
```

Note that this doesn't actually connect to Infobip platform,
it just creates a new instance of [`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC).
Connection is made via the [`connect`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#connect) method.
Before connecting, it is useful to set up event handlers, so you can perform something when the connection is set up,
when the connection is lost, etc.
Events are set up via [`on`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#on) method:

```javascript
infobipRTC.on('connected', function (event) {
    console.log('Connected with identity: ' + event.identity);
});
infobipRTC.on('disconnected', function (event) {
    console.log('Disconnected!');
});
```

Now you are ready to connect:

```javascript
infobipRTC.connect();
```

### Making a WebRTC call

You can call another WebRTC endpoint if you know their identity.
It is done via the [`callWebrtc`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-webrtc) method:

```javascript
let webrtcCall = infobipRTC.callWebrtc('Alice');
```

Or if you want to initiate a call with video:

```javascript
let webrtcCall = infobipRTC.callWebrtc('Alice', WebrtcCallOptions.builder().setVideo(true).build());
```

As you can see, the [`callWebrtc`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-webrtc) method
returns an instance of [`WebrtcCall`](https://github.com/infobip/infobip-rtc-js/wiki/WebrtcCall) as the result.
With it, you can track the status of your call and respond to events. Similar to the client,
you can set up event handlers, so you can do something when the called subscriber answers the call,
rejects it, the call is ended, etc. You set up event handlers with the following code:

```javascript
webrtcCall.on(CallsApiEvents.RINGING, function (event) {
    console.log('Call is ringing on Alice\'s device!');
});

webrtcCall.on(CallsApiEvents.ESTABLISHED, function (event) {
    console.log('Alice answered call!');
});

webrtcCall.on(CallsApiEvents.HANGUP, function (event) {
    console.log('Call is done! Status: ' + JSON.stringify(event.errorCode));
});

webrtcCall.on(CallsApiEvents.ERROR, function (event) {
    console.log('Oops, something went very wrong! Message: ' + JSON.stringify(event.errorCode));
});
```

The most important part of the call is definitely the media that travels between the subscribers.
It can be handled in an `ESTABLISHED` event where you have the remote media which can be streamed into your HTML page.
This is an example of how you can use it:

```javascript
<audio id="remoteAudio" autoplay/>

webrtcCall.on(CallsApiEvents.ESTABLISHED, function (event) {
    console.log('Alice answered call!');
    document.getElementById('remoteAudio').srcObject = event.stream;
});
```

At any time during the WebRTC call, users can add or remove their camera videos. In order to handle the video media, you
should set the event handlers for `CAMERA_VIDEO_ADDED`, `CAMERA_VIDEO_UPDATED` and `CAMERA_VIDEO_REMOVED` events.

```javascript
webrtcCall.on(CallsApiEvents.CAMERA_VIDEO_ADDED, function (event) {
    $('#localVideo').srcObject = event.stream;
});

webrtcCall.on(CallsApiEvents.CAMERA_VIDEO_UPDATED, function (event) {
    $('#localVideo').srcObject = event.stream;
});

webrtcCall.on(CallsApiEvents.CAMERA_VIDEO_REMOVED, function (event) {
    $('#localVideo').srcObject = null;
});
```

Then, you can turn on the camera using the 
[`cameraVideo`](https://github.com/infobip/infobip-rtc-js/wiki/WebrtcCall#camera-video) method.

```javascript
webrtcCall.cameraVideo(true);
```

Users can also start or stop sharing their screen. As was the case with camera video, first you should add handlers for
the following events: `SCREEN_SHARE_ADDED` and `SCREEN_SHARE_REMOVED`:

```javascript
webrtcCall.on(CallsApiEvents.SCREEN_SHARE_ADDED, function (event) {
    $('#localScreenShare').srcObject = event.stream;
});

webrtcCall.on(CallsApiEvents.SCREEN_SHARE_REMOVED, function (event) {
    $('#localScreenShare').srcObject = null;
});
```

Then, you can use the [`screenShare`](https://github.com/infobip/infobip-rtc-js/wiki/WebrtcCall#screen-share) method to 
start or stop sharing your screen.

```javascript
webrtcCall.screenShare(true);
```

In order to handle camera and screen share media from the other side, you should add event handlers for 
`REMOTE_CAMERA_VIDEO_ADDED`, `REMOTE_CAMERA_VIDEO_REMOVED`, `REMOTE_SCREEN_SHARE_ADDED` and 
`REMOTE_SCREEN_SHARE_REMOVED` events.

```javascript
webrtcCall.on(CallsApiEvents.REMOTE_CAMERA_VIDEO_ADDED, function (event) {
    $('#remoteCameraVideo').srcObject = event.stream;
});

webrtcCall.on(CallsApiEvents.REMOTE_CAMERA_VIDEO_REMOVED, function (event) {
    $('#remoteCameraVideo').srcObject = null;
});

webrtcCall.on(CallsApiEvents.REMOTE_SCREEN_SHARE_ADDED, function (event) {
    $('#remoteScreenShare').srcObject = event.stream;
});

webrtcCall.on(CallsApiEvents.REMOTE_SCREEN_SHARE_REMOVED, function (event) {
    $('#remoteScreenShare').srcObject = null;
});
```

Besides adding and removing camera and share screen, there are a few more things that you can do with the actual call.
One of them, of course, is to hang up.
That can be done via the [`hangup`](https://github.com/infobip/infobip-rtc-js/wiki/WebrtcCall#hangup) method on the call,
and after that, both parties will receive the `HANGUP` event upon hang up completion.

```javascript
webrtcCall.hangup();
```

You can simulate digit press during the call by sending DTMF codes (Dual-Tone Multi-Frequency).
This is achieved via [`sendDTMF`](https://github.com/infobip/infobip-rtc-js/wiki/Call#send-dtmf) method.
Valid DTMF codes are digits from `0-9`, `*` and `#`.

```javascript
webrtcCall.sendDTMF('*');
```

During the call, you can also mute (and unmute) your audio:

```javascript
webrtcCall.mute(true);
```

To check if the audio is muted, call the
[`muted`](https://github.com/infobip/infobip-rtc-js/wiki/Call#muted) method in the following way:

```javascript
let audioMuted = webrtcCall.muted();
```

Also, you can check the [`call status`](https://github.com/infobip/infobip-rtc-js/wiki/CallStatus):

```javascript
let status = webrtcCall.status();
```

Also, you can check information such as [`duration`](https://github.com/infobip/infobip-rtc-js/wiki/Call#duration),
[`start time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#start-time),
[`establish time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#establish-time) and
[`end time`](https://github.com/infobip/infobip-rtc-js/wiki/Call#end-time) by calling these methods:

```javascript
let duration = webrtcCall.duration();
let startTime = webrtcCall.startTime();
let establishTime = webrtcCall.establishTime();
let endTime = webrtcCall.endTime();
```

### Receiving a call

Besides making outgoing calls, you can also receive incoming calls.
In order to do that, you need to register the `incoming-webrtc-call` event handler of
[`InfobipRTC`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC) client.
That is where you define the behavior of the incoming call.
One of the most common things to do is to show Answer and Reject options on the UI.
For the purpose of this guide, let's look at an example that answers the incoming call as soon as it arrives:

```javascript
infobipRTC.on('incoming-webrtc-call', function (incomingWebrtcCallEvent) {
    const incomingCall = incomingWebrtcCallEvent.incomingCall;
    console.log('Received incoming call from: ' + incomingCall.source().identifier);

    incomingCall.on(CallsApiEvents.ESTABLISHED, function () {
    });
    
    incomingCall.on(CallsApiEvents.HANGUP, function () {
    });

    incomingCall.accept(); // or incomingCall.decline();
});
```

### Calling phone number

It is similar to calling the regular WebRTC user, you just use the
[`callPhone`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-phone) method instead of
[`callWebrtc`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-webrtc).
This method accepts an optional second parameter, where you define the `from` parameter.
Its value will be displayed on the calling phone device as the Caller ID.
The result of the [`callPhone`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-phone) is the
[`PhoneCall`](https://github.com/infobip/infobip-rtc-js/wiki/PhoneCall) with which you can do a few things such as
muting the call, hanging it up, checking its start time, answer time, duration and more. See reference 
[documentation](https://github.com/infobip/infobip-rtc-js/wiki/PhoneCall)
for a full list of available actions.

* Example of calling phone number with `from` defined:

```javascript
let phoneCall = infobipRTC.callPhone('41793026727', PhoneCallOptions.builder().setFrom('33712345678').build());
```

* Example of calling phone number without `from` defined:

```javascript
let phoneCall = infobipRTC.callPhone('41793026727');
```

### Calling Viber

Calling Viber is very similar to calling a phone number, except the call will arrive to the destination through
the Viber application. In order to call Viber, you can use the [`callViber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-viber) method,
which returns an instance of [`ViberCall`](https://github.com/infobip/infobip-rtc-js/wiki/ViberCall)

```javascript
let viberCall = infobipRTC.callViber('41793026727', '33712345678');
```

Please note that unlike in the [`callPhone`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-phone) method,
`from` is required and passed in as a second parameter to the [`callViber`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-viber) method.

### Room call

Room calls are types of calls that can have up to 15 participants.
The room call will start as soon as at least one participant joins.

Joining the room is done via the
[`joinRoom`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#join-room) method:

```javascript
let roomCall = infobipRTC.joinRoom('room-demo');
```

Or if you want to join the room with your video:

```javascript
let roomCall = infobipRTC.joinRoom('room-demo', RoomCallOptions.builder().setVideo(true).build());
```

As you can see, that method returns an instance of
[`RoomCall`](https://github.com/infobip/infobip-rtc-js/wiki/RoomCall) as the result.
With it, you can track the status of your room call,
do some actions (mute, share the screen, turn your camera on) and respond to events.

After the user successfully joined the room, the `ROOM_JOINED` event will be emitted.
It contains an id of the joined room, its name, participants and stream. You should implement an event handler for it, 
so that the user could be notified about joining the room, and the other participants might be shown to the user.

Here is an example of how to
handle [`room events`](https://github.com/infobip/infobip-rtc-js/wiki/RoomCall#on).

Let's assume that we have an audio HTML element with the id `roomAudio` and video HTML elements with the ids
`localVideo` and `localScreenShare`.

```javascript
roomCall.on(CallsApiEvents.ROOM_JOINED, function (event) {
    $('#remoteAudio').srcObject = event.stream;
    var participants = event.participants.map(participant => participant.endpoint.identifier).join(", ");
    console.log(`You have joined the room with ${event.participants.length} more participants: ${participants}`);
});

roomCall.on(CallsApiEvents.ROOM_LEFT, function (event) {
    console.log(`You have left the room with error code: ${event.errorCode.name}.`);
});

roomCall.on(CallsApiEvents.ERROR, function (event) {
    console.log(`Error! ${event.errorCode.name}`);
});

roomCall.on(CallsApiEvents.PARTICIPANT_JOINED, function (event) {
    console.log(`${event.participant.endpoint.identifier} joined the room call.`);
});

roomCall.on(CallsApiEvents.PARTICIPANT_LEFT, function (event) {
    console.log(`${event.participant.endpoint.identifier} left the room call.`);
});

roomCall.on(CallsApiEvents.PARTICIPANT_MUTED, function (event) {
    console.log(`${event.participant.endpoint.identifier} muted.`);
});

roomCall.on(CallsApiEvents.PARTICIPANT_UNMUTED, function (event) {
    console.log(`${event.participant.endpoint.identifier} unmuted.`);
});

roomCall.on(CallsApiEvents.CAMERA_VIDEO_ADDED, function (event) {
    $('#localVideo').srcObject = event.stream;
});

roomCall.on(CallsApiEvents.CAMERA_VIDEO_UPDATED, function (event) {
    $('#localVideo').srcObject = event.stream;
});

roomCall.on(CallsApiEvents.CAMERA_VIDEO_REMOVED, function (event) {
    $('#localVideo').srcObject = null;
});

roomCall.on(CallsApiEvents.SCREEN_SHARE_ADDED, function (event) {
    $('#localScreenShare').srcObject = event.stream;
});

roomCall.on(CallsApiEvents.SCREEN_SHARE_REMOVED, function (event) {
    $('#localScreenShare').srcObject = null;
});
```

The next two events are fired when another participant adds or removes the video.
You should implement these event handlers in order to add and/or remove an HTML video element with a media stream.

```javascript
roomCall.on(CallsApiEvents.PARTICIPANT_CAMERA_VIDEO_ADDED, function (event) {
    // add a new HTML video element with id remoteVideo-event.participant.endpoint.identifier
    $('#remoteVideo-' + event.participant.endpoint.identifier).srcObject = event.stream;
});

roomCall.on(CallsApiEvents.PARTICIPANT_CAMERA_VIDEO_REMOVED, function (event) {
    console.log(`Participant ${event.participant.endpoint.identifer} removed their camera`)
    // remove the HTML video element with id remoteVideo-event.participant.endpoint.identifier
});
```

The next two events are fired when another participant starts or stops sharing screen.
You should implement these event handlers in order to add and/or remove an HTML video element with a media stream.

```javascript
roomCall.on(CallsApiEvents.PARTICIPANT_SCREEN_SHARE_ADDED, function (event) {
    // add a new HTML video element with id remoteScreenShare-event.participant.endpoint.identifier
    $('#remoteScreenShare-' + event.participant.endpoint.identifier).srcObject = event.stream;
});

roomCall.on(CallsApiEvents.PARTICIPANT_SCREEN_SHARE_REMOVED, function (event) {
    // remove the HTML video element with id remoteScreenShare-event.participant.endpoint.identifier
});
```

When event handlers are set up and the room call is established,
there are a few things that you can do with it.

One of them, of course, is to leave the room. That can be done via the
[`leave`](https://github.com/infobip/infobip-rtc-js/wiki/RoomCall#leave) method at the room call.
Other participants will receive the `PARTICIPANT_LEFT` event upon leave completion.

```javascript
roomCall.leave();
```

During the room call, you can also mute (and unmute) your audio, by calling the
[`mute`](https://github.com/infobip/infobip-rtc-js/wiki/RoomCall#mute) method in the following way:

```javascript
roomCall.mute(true);
```

During the room call, you can start or stop sending your video, by calling the
[`cameraVideo`](https://github.com/infobip/infobip-rtc-js/wiki/RoomCall#camera-video) method in the following way:

```javascript
roomCall.cameraVideo(true | false);
```

After this method, the `CAMERA_VIDEO_ADDED` or `CAMERA_VIDEO_REMOVED` event is fired.

During the room call, you can start or stop sharing your screen, by calling the
[`screenShare`](https://github.com/infobip/infobip-rtc-js/wiki/RoomCall#screen-share) method in the following way:

```javascript
roomCall.screenShare(true | false);
```

After this method, the `SCREEN_SHARE_ADDED` or `SCREEN_SHARE_REMOVED` event is fired.

### Media Device information

Beside getting the information about one's media devices anytime after the user is registered to Infobip's platfom,
or during the call, you can get these infos using the static methods
in [`RTCMediaDevice`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice).

```javascript
let RTCMediaDevice = require('infobip-rtc');
```

For audio input devices you can use the static
[`RTCMediaDevice.getAudioInputDevices`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getAudioInputDevices)
method:

```javascript
RTCMediaDevice.getAudioInputDevices().then(
    mediaDevices => {
        mediaDevices.forEach(device => {
            console.log(device.label);
        });
    }
);
```

For audio output devices you can use the static
[`RTCMediaDevice.getAudioOutputDevices`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getAudioOutputDevices)
method:

```javascript
RTCMediaDevice.getAudioOutputDevices().then(
    mediaDevices => {
        mediaDevices.forEach(device => {
            console.log(device.label);
        });
    }
);
```

For video input devices you can use the static
[`RTCMediaDevice.getVideoInputDevices`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getVideoInputDevices)
method:

```javascript
RTCMediaDevice.getVideoInputDevices().then(
    mediaDevices => {
        mediaDevices.forEach(device => {
            console.log(device.label);
        });
    }
);
```

Also, you can get the stream of a specific device using the static
[`RTCMediaDevice.getMediaStream`](https://github.com/infobip/infobip-rtc-js/wiki/RTCMediaDevice#getMediaStream) method:

```javascript
let mediaDevices = await RTCMediaDevice.getAudioInputDevices();
let device = mediaDevices[0];
let stream = await RTCMediaDevice.getMediaStream(device.deviceId);
$('#audio').srcObject = stream;
//...
RTCMediaDevice.closeMediaStream(stream);
```

### Browser Compatibility

We support up to 5 most recent versions of these browsers (unless otherwise indicated):

|                         |                               | ![chrome](assets/icons/chrome.png)  | ![firefox](assets/icons/firefox.png) | ![safari](assets/icons/safari.png)  | ![edge](assets/icons/edge.png)      | ![opera](assets/icons/opera.png)  | ![opera](assets/icons/explorer.png) |
|------------------------:|------------------------------:|:----------------------------:|:-----------------------------:|:----------------------------:|:----------------------------:|:--------------------------:|:----------------------------:|
|                         |                               | **Google Chrome**            | **Mozilla Firefox**           | **Safari***                  | **Microsoft Edge****           | **Opera**                  | **Internet Explorer**        |
| **Android**             | ![android](assets/icons/android.png) | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **iOS*****                | ![ios](assets/icons/ios.png)         | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **Linux**               | ![linux](assets/icons/linux.png)     | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **macOS**               | ![macOs](assets/icons/mac.png)       | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |
| **Windows**             | ![windows](assets/icons/windows.png) | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)      | ![chrome](assets/icons/no.png)      | ![chrome](assets/icons/yes.png)     | ![chrome](assets/icons/yes.png)   | ![chrome](assets/icons/no.png)      |

\* WebRTC support in Safari started with version 11.

\** WebRTC support in Microsoft Edge for Android, iOS and macOS started with Chromium-based version 79.
InfobipRTC is supported only for Chromium-based Microsoft Edge versions for Windows also.

\*** WebRTC support in browsers other than Safari started with iOS version 14.3.

> **Note**: Mobile browsers are not able to receive calls and maintain call connectivity in the background.
> We recommend using [iOS](https://github.com/infobip/infobip-rtc-ios)
> and [Android](https://github.com/infobip/infobip-rtc-android) SDKs
> for creating mobile WebRTC Applications.
