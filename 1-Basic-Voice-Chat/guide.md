# Basic Voice Chat

Adding basic voice calling features to our website. Building from scratch using Vite.


### Basic App Setup

Create a new vite app.

`npm create vite@latest`

Basic Installs

```
cd your-app-name
npm install
npm run dev
```

Remove unnecessary files

- ~~javascript.jpg~~
- ~~counter.js~~
  
Clean out file contents

- **style.css** - Delete Everything inside
- **main.js** - Leave only the css import: `import './style.css'`
- **index.html** - Remove "app" div: `<div id="app"></div>`


Add in basic HTML:

```html
<div id="container">

     <div id="room-header">

        <h1 id="room-name"></h1>

        <div id="room-header-controls">
          <img id="mic-icon" class="control-icon" src="icons/mic-off.svg" />
          <img id="leave-icon" class="control-icon" src="icons/leave.svg" />
        </div>
      </div>

    <form id="form">
        <!-- <input name="displayname" type="text" placeholder="Enter display name..."/> -->
        <input type="submit" value="Enter Room"/>
    </form>

    <div id="members">

    </div>
</div>
```

Add Icons:

Inside `room-header-controls` in our HTML file we reference two icons:

- `mic-off.svg`
- `leave.svg`

You can find these files within the source code provided within the "icons" folder.


Add pre-built css

```css
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap');

:root {
  font-family: 'Roboto', sans-serif;
  font-size: 16px;
  color-scheme: dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #242424;
}

a {
  color: #646cff;
  text-decoration: none;
}
a:hover {
  color: #535bf2;
}

body {
  margin: 0;
}

#container{
  max-width: 800px;
  margin: 0 auto;
  padding: 1em;
}

#room-header{
  justify-content: space-between;
  align-items: center;
  padding: 1em 0;
  /* display: none; */
}

#room-header-controls{
  display: flex;
}

#room-header-controls > img {
  margin: 0 2px;
}

.control-icon{
    background-color: indianred;
    border: none;
    padding: 0.5em;
    height: 20px;
    cursor: pointer;
    border-radius: 5px;
}


#members{
  display: flex;
  flex-wrap: wrap;
}

.speaker{
  border:2px solid #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 120px;
  text-align: center;
  margin: 0.5em;
}

#avatars{
  display: flex;
  flex-wrap: wrap;
  margin: 1em 0;
}

.avatar-selection{
  height: 50px;
  width: 50px;
  object-fit: contain;
  border: 2px solid #FFF;
  border-radius: 50%;
  opacity: .5;
  cursor: pointer;
  margin: 0.25em;
}

.user-avatar{
  height: 75px;
  width: 75px;
  object-fit: contain;
  border: 2px solid #FFF;
  border-radius: 50%;
}

#form input{
  width: 100%;
  padding: 1em;
  margin-bottom:2em;
  box-sizing: border-box;
}

#form label{
  margin-bottom: 0.5em;
}

#form-fields{
  display: flex;
  flex-direction: column;
}
```

### Getting Started With Agora

Prerequisite's
- Have an account with agora.io
- Have an app ready: Ensure auth is set to APP ID only.

Install Agora RTC:

```
npm install agora-rtc-sdk-ng
```

**Configuring Agora RTC In Our App**

Inside `main.js` we will start by setting initial values for our app connection and RTC client.

1. Import the `AgoraRTC` class. This is the global entry point for all the methods provided by the Agora Web SDK. You can read more [here](https://api-ref.agora.io/en/voice-sdk/web/4.x/index.html#core-methods)
   
<br/>

2. Basic Credentials and user settings
   - `appid` - Your project app id from the agora console
   - `token` - Authentication token - Leave as `null` if you set the authentication mechanism to "APP ID only" when you created your app.
   - `rtcUid` - A user ID (UID) identifies a user in a channel. Each user in a channel should have a unique user ID. An RTC UID should be an integer (anything up to a 32 bit integer). In therory, this could be turned into a string but it is not recommended as it won't with certain agora products such as cloud recording. 
   - `roomId` - Agora requires a "channel name" for creating separate rooms users can join. We will use the name `roomId`, and just hard code this value as `main`. We will make this dynamic later so users can join different rooms.

<br/>

3. `audioTracks` will store our local tracks when we join a channel and all remote users who join later. Remote users will be stored as a key-value pair and will be identified by their unique id.

<br/>

4. `rtcClient` - Our client object which will be set once we initiate and join a channel. This will be our entry point to all the methods we need to join & leave rooms, toggle our mic, listen for key events, etc.

```js
//1
import AgoraRTC from "agora-rtc-sdk-ng"

//2
const appid = "YOUR-APP-ID"
const token = null
const rtcUid =  Math.floor(Math.random() * 2032)

let roomId = "main"

//3
let audioTracks = {
  localAudioTrack: null,
  remoteAudioTracks: {},
};

//5
let rtcClient;
```


**Initiate RTC Client & Join Channel**


The `initRtc` method will be responsible for the core configuration we need for joining a channel


1. Initiate client and join channel
    - `createClient` - initiates our RTC client and requires that we set the mode (live OR rtc) and encoding config
    - `join` - Creates or joins a room with our credentials (App ID, Room Name, token and user UID)

<br/>

2. Getting Mic Audio Tracks
   - `createMicrophoneAudioTrack` gets our local audio tracks which we will assign to the `localAudioTrack` key in our `audioTracks` object.

<br/>

3. Adding User To Dom - Create an HTML element and add it to dom with UID value. For now, this is the only thing we have to identify the user. We will add user names and avatars later.

`main.js`

```js

const initRtc = async () => {
  //1
  rtcClient = AgoraRTC.createClient({ mode: "rtc", codec: "vp8" });
  
  await rtcClient.join(appid, roomId, token, rtcUid)

  //2
  audioTracks.localAudioTrack = await AgoraRTC.createMicrophoneAudioTrack();
  await rtcClient.publish(audioTracks.localAudioTrack);

  //3
  document.getElementById('members').insertAdjacentHTML('beforeend', `<div class="speaker user-rtc-${rtcUid}" id="${rtcUid}"><p>${rtcUid}</p></div>`)
}
```

**Enter Room**

We want to join a room by initiating the `initRtc` method when our form is submitted. When this event fires we want to hide the form (button) and display our header so the "leave" button is visible

`main.js`


```js
let lobbyForm = document.getElementById('form')

const enterRoom = async (e) => {
  e.preventDefault()
  initRtc()

  lobbyForm.style.display = 'none'
  document.getElementById('room-header').style.display = "flex"
}

lobbyForm.addEventListener('submit', enterRoom)
```

**Leave Room**

When a user clicks the leave icon/button we want to stop all local audio tracks and leave the stream.

1. Stop and Close Tracks
   - `stop` - Stops playing the media track.
   - `close` - Closes a local track and releases the audio and video resources that it occupies. Once you close a local track, you can no longer reuse it.

2. Unpublish Tracks and Leave Channel
   - `unpublish` Unpublishes the local audio and/or video tracks. Calling this method will trigger the AgoraRTCClient.on("user-unpublished")  event for all remote users in the channel
   - `leave` - Leaves a channel. Calling this method will trigger the AgoraRTCClient.on("user-left")  event for all remote users in the channel

3. Remove members from DOM, display the form again and hide the header so the "leave" button is not visible.

```js

let leaveRoom = async () => {
  //1
  audioTracks.localAudioTrack.stop()
  audioTracks.localAudioTrack.close()

  //2
  rtcClient.unpublish()
  rtcClient.leave()

  //3
  document.getElementById('form').style.display = 'block'
  document.getElementById('room-header').style.display = 'none'
  document.getElementById('members').innerHTML = ''
}

document.getElementById('leave-icon').addEventListener('click', leaveRoom)
```


**Remote User Event Listeners**

Joining a channel and publishing a stream doesn't mean that other users in the same channel will begin to hear or see your stream. Before we can get to that point we need to create event listeners that listen for key channel events like when a user joins the channel, publishes, and unpublishes their stream leaves the stream. 

When these events occur we will respond by subscribing/unsubscribing to their audio tracks and playing them locally.

They can events we will listen for are:
- `user-joined` --> triggered when a remote user calls the `join()` event.
- `user-published` --> triggered when a remote user calls the `publish()` method.
- `user-left` --> triggered when a remote user calls the `leave()` method.

When these callbacks are called we want to trigger functions on the receiving end that respond. We use the `on` method which the client object gives us.

Inside the `initRtc` function let's add these event listeners, then create the corresponding functions:

```js
const initRtc = async () => {
  //...
  rtcClient.on('user-joined', handleUserJoined)
  rtcClient.on("user-published", handleUserPublished)
  rtcClient.on("user-left", handleUserLeft);
  //...
}
```

Next, we need to create the 3 functions that fire off when these callbacks are triggered:

1. `handleUserJoined` - will fire off when a new user joins. We will have access to the remote user object as a parameter. To better understand what the user object is comprised of try adding `console.log("USER:", user)` to see the object.

2. Create a new HTML element and add it to the dom so we can visually represent the joined user. To uniquely identify this user we will set the ID by accessing the `user.uid` attribute and also passing this into the paragraph tags.

3. `handleUserPublished` fires when a remote user publishes their audio and or video tracks. This method will give us access to the user object and the `mediaType` which will either be "audio" or "video".

4. When a new user publishes their track the first thing we want to do is call the `subscribe` method so we can see and or hear them.

5. Once we are subscribed to their audio/video tracks we want to add these tracks to our remote users' key in `audioTracks` and then `play()` the track.

6. When a user calls the `leave()` method we want to remove their audio tracks from our local state and remove them from the dom. We find the user in the dom by the specific user id we passed in to the HTML element

```js
  //1 
let handleUserJoined = async (user) => {
  //2
  document.getElementById('members').insertAdjacentHTML('beforeend', `<div class="speaker user-rtc-${user.uid}" id="${user.uid}"><p>${user.uid}</p></div>`)
} 

//3 
let handleUserPublished = async (user, mediaType) => {
  //4
  await  rtcClient.subscribe(user, mediaType);
  
  //5
  if (mediaType == "audio"){
    audioTracks.remoteAudioTracks[user.uid] = [user.audioTrack]
    user.audioTrack.play();
  }
}

//6
let handleUserLeft = async (user) => {
  delete audioTracks.remoteAudioTracks[user.uid]
  document.getElementById(user.uid).remove()
}

```
