# Displaying User Names

Passing Attributes through users to display user names and other forms of data.

### Adding And Retrieving Attributes

The reason we added the RTM SDK to our app was to make use of the extra functionality it gives us. One of those features is the ability to add attributes to our user which can be retrieved on the other end from remote users. 

You may recall that the reason we are only outputting our UID for each user is that the RTC uid only provides a limited amount of attributes to work with and not a way to modify the data passed through.

With the RTM SDK we can add & retrieve attributes.

To add we use the `addOrUpdateLocalUserAttributes` method and give it an object with the values we want to add:

```js
rtmClient.addOrUpdateLocalUserAttributes({'key1':'value', 'key2':'value'})
```

To retrieve these attributes we use the `getUserAttributesByKeys` method.

We call the method by passing in the UID of the member and an array of the methods we want. We can then destructure the response to get the values.

```js
const {key1, key2} = await rtmClient.getUserAttributesByKeys(MemberId, ['key1', 'key2'])
```

### Setting & Retrieving Display Names

Using what you just learned let's jump to the `initRtm` method and set the name attribute after we join a room. 

We will set the value of "name" using our form in the next step.

```js
let initRtm = async (name) => {
    await rtmClient.addOrUpdateLocalUserAttributes({'name':name})
}
```

**Setting Display Name**

Let's jump into our HTML file really quickly and first uncomment the "displayname" field we created earlier and make it a required field so users cant join without a name.


```html
<form id="form">
    <input required name="displayname" type="text" placeholder="Enter display name..."/>
    ...
</form>
```

Now inside of the `enterRoom` method before we call the `initRtm` method, lets set the value of the `displayName` variable to the form input field.

```js
const enterRoom = async (e) => {
  ...

  let displayName = e.target.displayname.value;
  initRtm(displayName)
  ...
}
```

This name will now be passed through into the custom user attribute we set.

**Retrieving Users Display Name**

Now on the receiving end lets make sure that all remote users can see this name.

Let's first jump to `handleMemberJoined` and get the name of the user that just joined.

After you call `getUserAttributesByKeys` make sure to update the paragraph tag to display the name: `<p>${name}</p>`

```js
let handleMemberJoined = async (MemberId) => {

  let {name} = await rtmClient.getUserAttributesByKeys(MemberId, ['name'])

  let newMember = `
  <div class="speaker user-rtc-${'---'}" id="${MemberId}">
      <p>${name}</p>
  </div>`

  document.getElementById("members").insertAdjacentHTML('beforeend', newMember)

}
```

Let's go ahead and do the same in the `getChannelMemebers` function.

```js
let getChannelMembers = async () => {
  let members = await channel.getMembers()
  for (let i = 0; members.length > i; i++){
    let {name} = await rtmClient.getUserAttributesByKeys(members[i], ['name'])

    let newMember = `
    <div class="speaker user-rtc-${'---'}" id="${members[i]}">
        <p>${name}</p>
    </div>`

  
    document.getElementById("members").insertAdjacentHTML('beforeend', newMember)
  }
}
```

### Passing Through More Attributes

Ok so now that we know how to pass in custom attributes we need to pass in the user's rtcUid along with their name. We need this value because of how the volume indicator highlights speaking users.

To summarize, the volume indicator looks for a class that starts with `user-rtc-` and ends with the users RTC UID. To make this work we will pass in this uid, retrieve it and set it in the class name so we can re-activate our volume indicator.

**Pass Users RTC UID As Attribute**

Back in the `initRtm` method lets add in one more attribute to our users.

Let's give the key the name of `userRtcUid` and turn the uid value into a string since we cannot pass integers here.

```js
let initRtm = async (name) => {
    ...
    await rtmClient.addOrUpdateLocalUserAttributes({'name':name, 'userRtcUid':rtcUid.toString()})
    ...
}
```
Now on the receiving end lets get this value in the `getChannelMembers` & `handleMemberJoined` functions.

After getting the values make sure to update the `user-rtc-` class with this new id: `user-rtc-${userRtcUid}"`

```js
let getChannelMembers = async () => {
  ...
    let {name, userRtcUid} = await rtmClient.getUserAttributesByKeys(members[i], ['name', 'userRtcUid'])

    let newMember = `
    <div class="speaker user-rtc-${userRtcUid}" id="${members[i]}">
        <p>${name}</p>
    </div>`
    ...
  
}
```

```js

let handleMemberJoined = async (MemberId) => {

  let {name, userRtcUid} = await rtmClient.getUserAttributesByKeys(MemberId, ['name', 'userRtcUid'])

  let newMember = `
  <div class="speaker user-rtc-${userRtcUid}" id="${MemberId}">
      <p>${name}</p>
  </div>`

  ...
}
```

Now you can go ahead and uncomment the `initVolumeIndicator` instance inside of `initRtc`

```js
const initRtc = async () => {
  ...
  initVolumeIndicator()
}
```
