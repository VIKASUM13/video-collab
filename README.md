# 100ms Sample React App

This is an example web app to demo 100ms' web SDK

## Prerequisites

You will need [Node.js](https://nodejs.org) version v12.13.0 or greater installed on your system

## Setup

Get the code by cloning this repo using git

```
git clone git@github.com:100mslive/sample-app-web.git
```

Once cloned, open the terminal in the project directory, and install dependencies with:

```
npm install
```

Create a new file `.env` and copy the values from `example.env`

```
cp example.env .env
```

### Token generation

Host your token generation service [following this guide](https://100ms.gitbook.io/100ms/helpers/runkit)

Update the `TOKEN_ENDPOINT` in `.env` file with your token generation service endpoint (eg. `https://ms-services-vjml47hk9gmu.runkit.sh/`)

### [Optional] Firebase config

To turn on the remote-mute feature, update the following values in `.env` file from your firebase project settings:

```
# Firebase config
FIREBASE_API_KEY=<firebaseConfig.apiKey>
FIREBASE_AUTH_DOMAIN=<firebaseConfig.authDomain>
FIREBASE_DATABASE_URL=<firebaseConfig.databaseURL>
FIREBASE_PROJECT_ID=<firebaseConfig.projectId>
FIREBASE_STORAGE_BUCKET=<firebaseConfig.storageBucket>
FIREBASE_MESSAGING_ID=<firebaseConfig.messagingSenderId>
FIREBASE_APP_ID=<firebaseConfig.appId>
```

Then start the app with:

```
npm start
```

The app should now be up and running at http://localhost:8080 🚀

![Screenshot](/public/screenshot.png?raw=true)

# SDK Documentation

This guide provides an overview of the key objects you'll use with 100ms' JavaScript SDK to build a live audio/video application.

## Supported Devices

| Platform | Chrome | Firefox | Opera | Safari |
|----------|--------|---------|-------|--------|
|Android 4.4 or later| Version 66 or later |Version 66 or later | Version 45 or later| No
|MacOS 10 or later| Version 66 or later | Version 66 or later |Version 45 or later |Version 11 or later | 
|Windows 7 or later | Version 66 or later | Version 66 or later | Version 45 or later |No |
| iOS  |No  |No |No |Version 12 or later|

## Concepts

- `Room` - A room represents a real-time audio, video session, the basic building block of the 100mslive Video SDK
- `Stream` - A stream represents real-time audio, video streams that are shared to a room. Usually, each stream contains a video track and an audio track (except screenshare streams, which contains only a video trac)
- `Track` - A track represents either the audio or video that makes up a stream
- `Peer` - A peer represents all participants connected to a room. Peers can be "local" or "remote"
- `Publish` - A local peer can share its audio, video by "publishing" its tracks to the room
- `Subscribe` - A local peer can stream any peer's audio, video by "subscribing" to their streams
- `Broadcast` - A local peer can send any message/data to all remote peers in the room.

## Pre-requisites

### 1. Get the 100ms JavaScript SDK

`npm install --save @100mslive/hmsvideo-web@latest`

### 2. Get Access Keys

Sign up on https://dashboard.100ms.live/register & visit `Developer` tab to get your access credentials

### 3. Generate a server-side token

To generate a server-side token, follow the steps described here - https://docs.100ms.live/server-side/generate-server-side-token

### 4. Create a room

To create a room, follow the steps described here - https://docs.100ms.live/server-side/create-room

### 5. Generate a client-side token

To generate a client-side token, follow the steps described here - https://docs.100ms.live/server-side/authentication

## Import modules & instantiate 100ms Client (HMSClient)

```import { HMSPeer, HMSClientConfig, HMSClient, LocalStream } from "@100mslive/hmsvideo-web';

const peer = new HMSPeer(userName:"<userName here>",authToken:"<authToken here>")

const config = new HMSClientConfig({
    endpoint: "wss://prod-in.100ms.live"
})

const client = new HMSClient(peer, config)

```

> authTokenis the client-side token generated by your token generation service.

## Connect to 100ms' server

After instantiating `HMSClient`, connect to 100ms' server

```
try {
    await client.connect()
} catch(err) {
    // Handle error
}
```

## Setup listeners

Add listener functions to listen to peers joining, establishing a connection to the server, peers publishing their streams etc.

```
client.on('connect',() => {
    // This is where we can call `join(room)`
});

client.on('disconnect', () => {});

client.on('peer-join', (room, peer) => {
    // Show a notification or toast message in the UI
});

client.on('peer-leave', (room, peer) => {
    // Show a notification or toast message in the UI
});

client.on('stream-add', (room,  peer, streamInfo) => {
    // subscribe to the stream if needed
});

client.on('stream-remove', (room, peer, streamInfo) => {
    // Remove remote stream if needed
});

client.on('broadcast', (room, peer ,message) => {
    // Show a notification or update chat UI
});

client.on('disconnected', () => {
    // If there is a temporary websocket disconnection, then execute code
    // to re-publish and subscribe all streams. eg. location.reload();
});
```

> Always wait for `connect` message listener after creating client before subscribing/publishing any streams.

> If say, 4 streams were already published when client connects to the room, then client receives `stream-add` messages for all those 4 streams as soon as client joins

> Remember to add `disconnected` message handler. Temporary websocket disconnections are common and trying to reconnect on disconnection will ensure the user sees the conference continuing instead of freezing up



## Join a room

```
try {
    await client.join(roomId);
} catch(err) {
    // Handle error
}
```

> This roomId should be generated using `createRoom` API

## Get a local stream

This method prompts the user for permission to use a media input which produces audio/video tracks such as a camera, screen, microphone

### Connect to default devices

```
const localStream = await client.getLocalStream({
    resolution: "vga",
    bitrate: 256,
    codec: "VP8",
    frameRate: 20,
    shouldPublishAudio:true,
    shouldPublishVideo:true
});
```

### Connect to specific device

In order to connect to a specific camera/mic, you can use the advancedMediaConstraints key which accepts browser's native [MediaStreamConstraints](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamConstraints) as shown below. To get deviceIDs, use [enumerateDevices](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/enumerateDevices)

```
const localStream = await client.getLocalStream({
		resolution: "vga", //This defines the video height and width. Can be qqvga, qvga, shd, hd
		bitrate: 256, //This is the maximum bitrate to cap the video at 
		codec: "VP8",
		frameRate: 20,
		shouldPublishAudio:true,
		shouldPublishVideo:true,
		advancedMediaConstraints: {
				video: {
						deviceId: "e82934fe80bdd62ed2aac541f5fd53e53d98abb0b738c6f52edea4f5014d32d8"
				},
				audio: {
						deviceId: "756814e591e116616c740e39b307a8015cac4c2511950e0240cf7fbe62736dfd"
				}
		}
});
```

> For advanced use cases: all `stream` objects returned by `getLocalStream` extends browser's native MediaStream class and implements all its [methods](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream)

> The settings above are recommended settings for most use cases. You can increase resolution to hd and bandwidth to 1024 to get higher quality video.

## Get local media for screenshare

This method prompts the user for permission to share their screen and choose the screnshare source

```
const localScreen = await client.getLocalScreen({
    bitrate: 0,
    codec: "VP8",
    frameRate: 10,
});

//If your primary usecase is sharing text

localScreen.getVideoTracks().forEach(track => {
    if ('contentHint' in track) {
        track.contentHint = 'text';
    }
});
```

## Display local stream

All `stream` objects can be attached to HTML video elements. eg. The local stream from the user's camera

```
//This is React implementation. 
//Replace ref, useRef with id, getElementById for native HTML implementation.

//Create a reference for the video element to which the local stream will be attached
const localVideo = useRef();

//Attach local stream to the video element
localVideo.current.srcObject = local;


//Add video element
<video autoPlay muted ref={localVideo}>
</video>
```

> Remember to set `muted` to `true` and mirror the local webcam stream

## Publish local stream to room

```
try {
    await client.publish(local, roomId);
} catch(err) {
    // handle the error
}
```

> A client can publish multiple streams eg. a screenshare, an in-built webcam and an external webcam all together


## Subscribe to a remote peer's stream

This method "subscribes" to a remote peer's stream. This should ideally be called in the `stream-add` message listener.

```
try {
    const remote = await client.subscribe(mid, roomId);
    // Do something with remote stream
} catch(err) {
    // Handle error
}
```

## Unsubscribe to a peer's stream

```
try {
    await client.unsubscribe(remoteStream, roomId);
} catch(err) {
    // Handle error
}
```

## Broadcast a payload to the room

```
try {
    await client.broadcast(payload, roomId);
} catch(err) {
    // Handle error
}
```

## Disconnect client

```
client.disconnect();
```

## Mute/unmute local video/audio

100ms SDK's `Stream` interface has `mute` and `unmute` methods provided to mute and unmute video or audio respectively.

```
// To mute local stream audio
local.mute('audio')

// To unmute local stream audio
local.unmute('audio')

// To mute local stream video
local.mute('video')

// To unmute local stream video
local.unmute('video')
```

## Change quality of audio/video mid-stream

You can use `applyConstraints` to change the quality/source of the video mid-stream.

### Change quality

```
client.applyConstraints({
      bitrate: 0,
      codec: "VP8",
			resolution:"hd",
      frameRate: 10,
},
localStream)
```

Refer the full SDK Documentation here - https://docs.100ms.live/client-side/web








