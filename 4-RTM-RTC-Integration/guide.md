# RTM-RTC Integrations

In order to add more functionality to our app like displaying user names & avatars, we need to configure our app to use the agoras RTM SDK and re-think our approach a little.

Agora RTC takes care of transmiting audio and video data, and it does that will. Using the Agora RTM (Signaling) we will be able to create a room chat room enviroment with settings to pass data back and fourth as needed, this can be for displaying the room participants, number of participants in a room, sending group or peir to peir messages and more.

### Comparing RTC and RTM (Signaling) Events and Methods

**Client Instance**

```js
//RTC
rtcClient = AgoraRTC.createClient({ mode: "rtc", codec: "vp8" });

//RTM
rtmClient = AgoraRTM.createInstance(appid)
await rtmClient.login({'uid':rtmUid, 'token':token})
```

**Joining Channel**

```js
//RTC
await rtcClient.join(appid, roomId, token, rtcUid)

//RTM
channel = rtmClient.createChannel(roomId)
await channel.join()
```

**Join Channel Event Listener**

```js
//RTC
rtcClient.on('user-joined', handleUserJoined)

//RTM
channel.on('MemberJoined', handleMemberJoined)
```

**Leaving Channel**

```js
//RTC
audioTracks.localAudioTrack.stop()
audioTracks.localAudioTrack.close()

rtcClient.unpublish()
rtcClient.leave()

//RTM
channel.leave()
rtmClient.logout()
```

**Leave Channel Event Listener**

```js
//RTC
rtcClient.on("user-left", handleUserLeft);

//RTM
channel.on('MemberLeft', handleMemberLeft)
```

### Rethinking Our Approach.

Let's start by looking at some events and code that should be taken care of by agora rtm instead of rtc

**Remove handleUserJoined**

The only thing this event does is find the member that just joined and add them to the dom. The only problem here is that we only have the user uid here so we cant display anything more. Agoro RTM has a better version of this callback that gives us more data so let's just comment this one out for now:

```js
/*
let handleUserJoined = async (user) => {
  document.getElementById('members').insertAdjacentHTML('beforeend', `<div class="speaker user-rtc-${user.uid}" id="${user.uid}"><p>${user.uid}</p></div>`)
} 
*/
```

We can also comment out the callback listener for this method inside of `initRtc`

```js
const initRtc = async () => {
  ...
  //rtcClient.on('user-joined', handleUserJoined)
  ...
}
```

**Modify handleUserLeft**

When a user leaves we still want to delete their audio tracks so RTC will take care of that, but we want the RTM SDK to handle the actual DOM update so let's comment out the `remove` method

```js
let handleUserLeft = async (user) => {
  delete audioTracks.remoteAudioTracks[user.uid]
  //document.getElementById(user.uid).remove()
}
```

**Temporarily comment out the initiateVolumeIndicator method**

Because agora RTC will no longer take care of any DOM updates we should comment out the `initiateVolumeIndicator` call inside of `initRtc` to avoid any errors that come from our incomplete code. Don't worry, we'll turn this on again soon.

```js
const initRtc = async () => {
  ...
  //initVolumeIndicator()
  ...
}
```


**Remove Current User**

When we first join a room we have a method that gets our current local user and adds us to the dom. This will be done in a new way with Agora RTM so let's comment this out as well.

```js
const initRtc = async () => {
  ...
  //document.getElementById('members').insertAdjacentHTML('beforeend', `<div class="speaker user-rtc-${rtcUid}" id="${rtcUid}"><p>${rtcUid}</p></div>`)
  ...
}
```

> NOTE: At this point, everything should work as before beside any DOM updates you should still be able to join a room, toggle your mic and hear other users, but you won't see them.



### Integrating Agora RTM

Install Agora RTC SDK

```
npm install agora-rtm-sdk
```

Import AgoraRTM at the top of `main.js`

```js
import AgoraRTM from "agora-rtm-sdk"
```

Add RTM UID

An RTM UID must be a string, so let's use the same method that we used to create an RTC UID and create a rtmUid with the `string()` method.

```js
const rtmUid =  String(Math.floor(Math.random() * 2032))
```

RTM Client and Channel placeholder

With Agora RTM we need to first create a channel (AKA room), then join that channel with our rtmClient. For the room name we will use the same as we did with RTC, `roomId`.

```js
let rtmClient;
let channel;
```


**Initiating An Rtm Client and Channel**


1. The `initRtm` function will take in a user display name and will trigger our initial client configuration and channel login

2. Create rtm client instance (Using rtmUid and the same token as with rtc)

3. create and join channel

```js
//1
const initRtm = async (name) => {
  //2
  rtmClient = AgoraRTM.createInstance(appid)
  await rtmClient.login({'uid':rtmUid, 'token':token})

  //3
  channel = rtmClient.createChannel(roomId)
  await channel.join()
}
```


Call this method inside of the `enterRoom` method just underneath the initRtc method. We'll create a variable called `displayName` with no value for now and pass it into the initRtm method as a parameter.


```js
const enterRoom = async (e) => {
  ...
  //initRtc()
  let displayName;
  initRtm(displayName)
  ...
}
```

**Leaving Channel**

Unlike the RTC SDK we don't automatically leave a channel when we close our browser. So before we do anything else and run into errors lets create a method that leaves our RTM channel when we click the exit icon and when the dom unloads.

First let's create a method called `leaveRtmChannel`

Here we will first leave the channel, then log out our client.

```js
let leaveRtmChannel = async () => {
  await channel.leave()
  await rtmClient.logout()
}
```


Next, we want to call this method in the leave room function since we already have this configured with the on-click event for the exit icon. This should be placed just underneath the `rtcClient` leave method so we can first close out the rtc channel.

```js
let leaveRoom = async () => {
  ...
  leaveRtmChannel()
  ...
}
```

last, we need to consider what happens when a user closes their browser without clicking the "leave" icon. For this, all we need to do is add a `beforeunload` event listener to the window. We can add this at the bottom of the `initRtm` method.


```js
let initRtm = async (name) => {
  window.addEventListener('beforeunload', leaveRtmChannel)
}
```

**Handle Member Joined**

Agora RTM has a similar call-back method to the RTC `user-joined` event. Just like when a rtcClient joins, the `user-joined` callback is triggered, so does the `MemberJoined` callback get triggered when a rtmClient joins a channel. This is the RTM version of that same callback, only now we have more control over our user which you will see soon.


Create our `handleMemberJoined` method.

When this event is called we get access to the rtmUid of the remote user that just joined. We will use this uid to identify the user in the HTML when we add it to the dom.

> Note: Leave the "user-rtc-" class with "---" as the last part of the class for now. 

At this point, we only have access to the new member's rtmUid and not the rtcUid. So until we retrieve the value, just add dashes to the `user-rtc-` class.

```js
let handleMemberJoined = async (MemberId) => {

  let newMember = `
  <div class="speaker user-rtc-${'---'}" id="${MemberId}">
      <p>${MemberId}</p>
  </div>`

  document.getElementById("members").insertAdjacentHTML('beforeend', newMember)

}
```

To trigger this callback add the following at the bottom of the `initRtm` method

```js
let initRtm = async (name) => {
  ...
  channel.on('MemberJoined', handleMemberJoined)
  ...
}
```

**Handle Member Left**

Just like the handle member left function, we have an alternative to the 'user-left' callback from the RTC SDK. The Agora RTM version of this4 is called `MemberLeft`.

So let's first create our function.

All this function will do is find the users in the dom and remove them:

```js
let handleMemberLeft = async (MemberId) => {
  document.getElementById(MemberId).remove()
}
```

And not call it inside of `initRtm`:

```js
channel.on('MemberLeft', handleMemberLeft)
```

> NOTE: If you give this a test you'll notice that you only see the remote user and not the local user. You will also only see users that join after you joined. We will fix this next

**Retrieving All Members In A Channel**

When we first join a channel we want to display all users currently in that channel, including ourselves, then let the `MemberJoined` and `MemeberLeft` methods take care of updating the dom while we are in the room.

With Agora RTM we can call the `getMembers()` on the channels calls and this will return every active user in this channel.

After we call this method we will loop through every member and add them to the DOM.

Let's start by creating a function called `getChannelMemebers`.

1. Get all memebers with `channel.getMemebers()`

2. Loop through the response and create HTML element for each user and add them to the dom

```js
let getChannelMembers = async () => {
  //1
  let members = await channel.getMembers()

  //2
  for (let i = 0; members.length > i; i++){

    let newMember = `
    <div class="speaker user-rtc-${'-----'}" id="${members[i]}">
        <p>${members[i]}</p>
    </div>`
  
    document.getElementById("members").insertAdjacentHTML('beforeend', newMember)
  }
}
```

Now the last thing we need to do is call this function inside of `initRtm` right after we join the channel.

```js
let initRtm = async () => {
  ...
  getChannelMembers()
  ...
}
```
