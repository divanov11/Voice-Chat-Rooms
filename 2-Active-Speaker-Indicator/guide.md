# Active Speaker Volume Indicator

Adding active speaker indicators to view who is currently talking in a voice chat room.


### Create and Initialize Volume Indicator

To add active speaker recognition we have a method called `enableAudioVolumeIndicator` which reports the volume of each user in the channel. This is reported by triggering the `AgoraRTCClient.on("volume-indicator")` call back every two seconds. 

Checking the volume every two seconds is too long of a delay to get an active reading so we will use the `AgoraRTC.setParameter('AUDIO_VOLUME_INDICATION_INTERVAL')` method to customize the callback to trigger every 200ms. This will give us a more accurate reading

1. Customize the volume indicator with the `setParameter` method and then enable the indicator with `enableAudioVolumeIndicator`.

2. On each callback loop through each volume and console out the volume level and user id to visually see what's happening. What we are looking for is a reading above the resting rate. 
  
I found that a reading every `200ms` gives me a resting volume at about 30 for anything over 50 means someone is speaking. You should play with these settings and see what works best for you

3. Last thing we want to do with this indicator is get the current user's id. Let's find the HTML element that represents them in the dom, and either add a green border to their wrapper or set this back to its original color.

> NOTE: AgoraRTC.setParameter(...) is part of agoras internal API and not officially documented as the public docs/SDK so you will see a warning when you use it.


```js
let initVolumeIndicator = async () => {

  //1
  AgoraRTC.setParameter('AUDIO_VOLUME_INDICATION_INTERVAL', 200);
  rtcClient.enableAudioVolumeIndicator();
  
  //2
  rtcClient.on("volume-indicator", volumes => {
    volumes.forEach((volume) => {
      console.log(`UID ${volume.uid} Level ${volume.level}`);

      //3
      try{
          let item = document.getElementsByClassName(`user-rtc-${volume.uid}`)[0]

         if (volume.level >= 50){
           item.style.borderColor = '#00ff00'
         }else{
           item.style.borderColor = "#fff"
         }
      }catch(error){
        console.error(error)
      }


    });
  })
}
```

This method should be initialized at the bottom of the `initRtc` function.

```js
const initRtc = async () => {
  //...
  initVolumeIndicator()
}
```