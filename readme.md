# Voice Rooms Chat App

A simple voice chat app using the Agora RTC And RTM Web SDK. 

Watch the complete video course [here!](https://youtu.be/abka2ug0kbs)

**Room**
<img src="images/room-preview.png"/>

**Lobby**

<img src="images/lobby-preview.png"/>

### Installation

> Ensure you have Node JS installed


```
git clone https://github.com/divanov11/Voice-Chat-Rooms

cd Voice-Chat-Rooms/demo

npm install
```

Add you APP ID Inside of `appid.js`

> NOTE: Get this from your Agora console when you initiate a new project.

```js
//appId.js
const appid = "YOU AGORA APP ID"

export default appid;
```

Start Development server

```
npm run dev
```

### Features
- Create/Join breakout rooms
- Active speaker volume indicator
- Display user names and avatars

