# Mic Mute Toggle

Adding the ability to toggle mute and unmute during a conversation.


### Toggle Mic Method

Before getting started we want to add an initial state to our mic. 

**Mic Initial State**

At the top of our `main.js` file create a variable for our mic state and set it to `true`, as in the mic is muted.

```js
let micMuted = true
```

To set the initial state for the mic let's add the following inside of the `initRtc` method, just underneath the `createMicrophoneAudioTrack`. This will ensure that our mic is off when we first join a room:

```js
const initRtc = async () => {
  //..
  //audioTracks.localAudioTrack = await AgoraRTC.createMicrophoneAudioTrack();
  audioTracks.localAudioTrack.setMuted(micMuted)
  //..
}
```

**Toggle Mic Method**

This method will toggle our mic, our mic state, and the mic icon to show an active/inactive state.

The logic here is simple:

- When this method is triggered, if the mic is muted, update the mic icon image and color, change the state to reflect the opposite status, then call the `setMuted()` method.

`setMuted` takes the value of either true or false and changes the status of our audio.

We also want to add an event handler to trigger this function.


```js
const toggleMic = async (e) => {
  if (micMuted){
    e.target.src = 'icons/mic.svg'
    e.target.style.backgroundColor = 'ivory'
    micMuted = false
  }else{
    e.target.src = 'icons/mic-off.svg'
    e.target.style.backgroundColor = 'indianred'
    
    micMuted = true
  }
  audioTracks.localAudioTrack.setMuted(micMuted)
}

document.getElementById('mic-icon').addEventListener('click', toggleMic)
```
