# Setting up expo to receive push notifications using FCM and APNS with custom sound

**This guide is only meant to setup the client side, a react native app built with expo. It is not meant as guide on how to write server side code to send the notifications.**

### Step 1: install expo-notifications

First we will need to install expo-notifications as a dependancy in our expo project.

`expo install expo-notifications` in the root of your project

### Step 2: add any assets you would like

Add the sound files with `.wav` extension, and optionally notification icon( 96\*96 png all white) to your asset folder

### Step3: Edit app.json

Inside app.json, inside the **expo** object, create a **plugins** key with an object such as:

```
    "plugins": [
      [
        "expo-notifications",
        {
          "icon": "./assets/notification_icon.png",
          "sounds": ["./assets/notfication_sound.wav"],
          "color": "#E87EA1"
        }
      ]
    ]
```

You can it [documented](https://docs.expo.dev/versions/v45.0.0/sdk/notifications/#configurable-properties) by expo

### Step 4: create a new firebase project and download config files

- Go to the firebase [console](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjO7t_Dkuf4AhUGVvEDHXlzCPQQFnoECA8QAQ&url=https%3A%2F%2Fconsole.firebase.google.com%2F&usg=AOvVaw2FZlXJ-vssrAqr1uc6tr-x) and create a new project.

- add an android app, download google-services.json, save it the root of your project, add path to expo>android>googleServicesFile
- add an ios app, download GoogleServices-Info.plist, save it the root of your project, and add path to expo>ios>googleServicesFile

### Step 5: Build a custom dev client to test it out

In your terminal: run `eas build -p android -profile development`, let the eas cli guide you through it until you get a dummy app.

### Step 6: Add FCM server key

- navigate to : `https://expo.dev/accounts/<expo account name>/projects/<project name>/credentials/android/<android bundle identifier>`

- Scroll down to server credentials(FCM server key)

- Press on _Add a FCM Server Key_
  1. Open your project's firebase console again
  2. At the top of the sidebar, click the gear icon to the right of Project Overview to go to your project settings.
  3. Click on the Cloud Messaging tab in the Settings pane.
  4. Copy the token listed next to Server key.
     > Note: Server Key is only available in Cloud Messaging API (Legacy), which may be Disabled by default. Enable it by clicking the 3-dot menu > Manage API in Google Cloud Console and follow the flow there. Once the legacy messaging API is enabled, you should see Server Key in that section.
- Add as google cloud messaging token and save.
