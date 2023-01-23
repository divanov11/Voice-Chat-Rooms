# Creating Rooms

Creating dynamic from names from login form.

**Form room name**

The first thing we'll need to do is add a room name field to the login form. We will then take this field value and update our `roomId` variable. Currently, this is set to `let roomId = 'main'` and we want to change this so we can have users join more than one room.

For a little styling let's also add some `<label>` tags and a div around our form fields with the ID of `form-fields`.

```html
<form id="form">
    <div id="form-fields">
        <label>Display Name:</label>
        <input name="displayname" type="text" placeholder="Enter username..." required/>

        <label>Room Name:</label>
        <input name="roomname" type="text" placeholder="Enter room name..." required/>

        <input type="submit" value="Enter Room"/>
    </div>
</form>
```

**Setting new room name on submit**

Now when we submit our form, what we want to do is extract this `roomname` value from our form and set it as the `roomId`. This can be done in the `enterRoom` function just before we call the `initRtc` & `initRtm` functions.

```js
const enterRoom = async (e) => {
    e.preventDefault()

    roomId = e.target.roomname.value.toLowerCase();
    initRtc()
    initRtm(e.target.displayname.value)
    ...
}
```

And now we can update the initial `roomId` value to be `undefined` until the `enterRoom` function is call.

```js
let roomId;
```

**Getting and setting the room name in url**

A convenient way to share a room with another participant is to simply grab the url and send it over so they can join, especially if we have a long or hard-to-spell room name. 

In this step what we will do first is make sure that when we submit our first the room name gets added to the URL params.

We'll use the `replaceSate` method and append `?room=${roomId}` as the last parameter. This will update our url to reflect this value.

```js
const enterRoom = async (e) => {
  ...

  roomId = e.target.roomname.value.toLowerCase();
  window.history.replaceState(null, null, `?room=${roomId}`);
 ...
}
```

The next step here would be to ensure that we are retrieving what's in the URL parameter if there is a value there. We'll create a function that will check this value and set its return value to the initial `roomId` value.

Let's create this function at the top of our `main.js` just above where we set our inital `roomId` variable.

This method will extract what's in the URL and set it to that value or just leave it as null.

```js
const getRoomId = () => {
  const queryString = window.location.search;
  const urlParams = new URLSearchParams(queryString);

  if (urlParams.get('room')){
    return urlParams.get('room').toLowerCase()
  }
}

let roomId = getRoomId() || null
```

The last thing to do is to make sure that if there is a room parameter in the URL that it gets set in the form field as the initial value on load. We can add this just under the `roomId` variable when it's set.

```js
...
let roomId = getRoomId() || null
document.getElementById('form').roomname.value = roomId
...
```
