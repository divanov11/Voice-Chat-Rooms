# Displaying Avatars

Adding and displaying user avatars.

**Add Avatars to Form**

Before we do anything here we need to add our avatars to our form so users can select them. In the real world, we would have a way of uploading our own images or have some kind of authentication system that remembers our profile picture but this will do for now.

Attached in the source code in section #7 is a folder called "avatars" with some images in there so if you want to use that go ahead and bring that into your project.

Using these images let's go ahead and add the following inside of our HTML form, at the top just above the form-fields div.

```html
<form id="form">
  <div>
    <h3>Select An Avatar:</h3>
  </div>
  <div id="avatars">
      <img class="avatar-selection" src="avatars/male-1.png"/>
      <img class="avatar-selection" src="avatars/male-2.png"/>
      <img class="avatar-selection" src="avatars/male-4.png"/>
      <img class="avatar-selection" src="avatars/male-5.png"/>

      <img class="avatar-selection" src="avatars/female-1.png"/>
      <img class="avatar-selection" src="avatars/female-2.png"/>
      <img class="avatar-selection" src="avatars/female-4.png"/>
      <img class="avatar-selection" src="avatars/female-5.png"/>
  </div> 

  <div id="form-fields">...</div>
  ....
```

**Selecting Avatars**

To select an avatar we will need to add an event listener to each image in our `avatars` wrapper. On click, we will highlight the avatar and use its `src` attribute to assign it to the `avatar` variable we are about to create.

Let's first set this variable at the top of our `main.js` file

```js
let avatar = null;
```

Now let's loop over our collection of HTML elements (avatar-selection) and add our event listener.

In the event listener, we will add a function that adds the selected image `src` value to the value our our `avatars` variable and also change the border color and opacity to represent the selected image.

```js
const avatars = document.getElementsByClassName('avatar-selection')

for (let i=0; avatars.length > i; i++){
  
  avatars[i].addEventListener('click', ()=> {
    for (let i=0; avatars.length > i; i++){
      avatars[i].style.borderColor = "#fff"
      avatars[i].style.opacity = .5
    }

      avatar = avatars[i].src
      avatars[i].style.borderColor = "#00ff00"
      avatars[i].style.opacity = 1
  })
}
```

To ensure that a user has selected an avatar before joining a room let's check the value of the avatar variable at the start of the `enterRoom` function. If no avatar has been selected we will simply call the `alert` function notifying the user and then call the return method to stop the rest of the function from running.

```js
if (!avatar){
  alert('Please select an avatar')
  return
}
```

**Passing Avatar Value to User**

Now that the avatar has been selected, the easy part is passing this as another value to the `addOrUpdateLocalUserAttributes` method and retrieving it on the other end with `getUserAttributesByKeys`.

Let's add the value and then retrieve it in the `getChannelMemebers` and `handleMemeberJoined` method.

Add attribute:
 
```js
//initRtm
await rtmClient.addOrUpdateLocalUserAttributes({'name':name, 'userRtcUid':rtcUid.toString(), 'userAvatar':avatar})
```

Retrieve attribute - Once the remote avatar is retrieved, add an `img` tag to the `speaker` wrapper with `userAvatar` as the `src` value and `avatar-${userRtcUid}"` in the image class. 

```js
//getChannelMemebrs & handleMemeberJoined
let {name, userRtcUid, userAvatar} = await rtmClient.getUserAttributesByKeys(MemberId, ['name', 'userRtcUid', 'userAvatar'])

let newMember = `
  <div class="speaker user-rtc-${userRtcUid}" id="${MemberId}">
    <img class="user-avatar avatar-${userRtcUid}" src="${userAvatar}"/>
      <p>${name}</p>
  </div>`
```

**Highlighting image in volume indicator**

Currently, we have a border around our entire speaker wrapper and are highlighting the entire wrapper when the volume indicator hears a noise. The effect we want to achieve is to highlight just the image and not have a border at all around the speaker. 

So let's remove the `border`from our CSS file, then update the volume indicator method to look for the `avatar-${volume.uid}` class instead of `user-rtc-${volume.uid}.`

```css
.speaker{
  /* border:2px solid #fff;*/
  ...
}
```


```js
let initVolumeIndicator = async () => {
  ...
  let item = document.getElementsByClassName(`avatar-${volume.uid}`)[0]
  ...
}
```

**Displaying Room Name**

The last thing we need to do is add a little style to our room by displaying the room name inside of our `room-name` element. This can be added to the `enterRoom` method.


```js
document.getElementById('room-name').innerText = roomId
```
