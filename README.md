# Setting up an expo managed project (not applicable to expo go) to receive push notifications using FCM with custom sound on IOS and Android

## Attention:

-   You can not follow this guide if you use Expo GO
-   This is tested against a custom development client with managed workflow

## Why would you go through this hard setup instead of simple Expo Push API:

-   You are migrating an app to react-native that already has the server side configured to send push notifications using FCM
-   You would like to use custom notification sounds which for now is not covered by Expo Push API as per their [docs](https://docs.expo.dev/push-notifications/sending-notifications/#message-request-format) as of 8 Jul 2022
-   You would like to fine tune the notification more than what Expo Push API provides

This guide is only meant to explain the setup needed on both android and ios to receive push notifications using Firebase Cloud Messaging. This guide replace relying on Expo to send your notifications. Even though we will not be using Expo Push Api to send notifications, we will be using expo's notification library, and eas services, which I am very thankful for and apprieciate their hard work.

While using Expo push api is simple and convenient, you may need this guide for the following reasons:

### Step 1: Install expo-notifications

First we will need to install expo-notifications as a dependancy in our expo project.

`expo install expo-notifications` in the root of your project

### Step 2: Add assets

You can add two assets that are related to notifications:

-   sounds
    -   recommended `.wav` sound files as per Expo [docs](https://docs.expo.dev/versions/v45.0.0/sdk/notifications/#configurable-properties)
-   icons
    -   size: 96\*96
    -   png format
    -   all white with transparency
        -   This medium [blog](https://proandroiddev.com/android-iconography-notification-3acd3e00d6ec) has good explanation of android notification icons

To follow along with this guide, add a `notification_sound.wav` and `notification_icon.png` to your assets folder

### Step 3: Edit app.json

In order for the assets in **step 2** to be bundled we will use the config plugin provided by `expo-notifications` package we installed in **step 1**

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

You can refer to the [documentation](https://docs.expo.dev/versions/v45.0.0/sdk/notifications/#configurable-properties) by expo

### Step 4: Create Firebase project and Install config files

-   Go to the firebase [console](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjO7t_Dkuf4AhUGVvEDHXlzCPQQFnoECA8QAQ&url=https%3A%2F%2Fconsole.firebase.google.com%2F&usg=AOvVaw2FZlXJ-vssrAqr1uc6tr-x) and create a new project.

-   add an android app
    1. download google-services.json
    2. save it in the root of your expo project
    3. add the path to google-services.json as a value for expo.android.googleServicesFile in app.json
-   add an ios app
    1. download GoogleServices-Info.plist
    2. save it in the root of your expo project
    3. add the path to GoogleServices-Info.plist as a value for expo.ios.googleServicesFile in app.json

### Step 5: Ios SDK setup and configuration

1. Go to the Firebase console
2. Click the gear icon on the top right > project settings > General
3. Scroll down to **Your apps** section and select ios

    > Make sure you fill out the bundle id and Team Id

4. Navigate to Cloud Messaging from the top bar and scroll down to Apple App configuration
5. Make sure you add in the APNs Authentication key(should have an apple developer account)

### Step 6: Build

I am using eas services by expo, but you are free to build it locally

In your terminal: run `eas build -p android -profile development`, let the eas cli guide you through it until you get a dummy app.

### Step 7: Add FCM server key

-   navigate to : `https://expo.dev/accounts/<expo account name>/projects/<project name>/credentials/android/<android bundle identifier>`

-   Scroll down to server credentials(FCM server key)

-   Press on _Add a FCM Server Key_
    1. Open your project's firebase console again
    2. At the top of the sidebar, click the gear icon to the right of Project Overview to go to your project settings.
    3. Click on the Cloud Messaging tab in the Settings pane.
    4. Copy the token listed next to Server key.
        > Note: Server Key is only available in Cloud Messaging API (Legacy), which may be Disabled by default. Enable it by clicking the 3-dot menu > Manage API in Google Cloud Console and follow the flow there. Once the legacy messaging API is enabled, you should see Server Key in that section.
-   Add as google cloud messaging token and save.

### Step 8: Write a basic app to give push token

Just code a simple app that requests for the device push token and displays it:

```
  export default function App() {
  const [devicePushToken, setDevicePushToken] = useState<string | null>(null);

  useEffect(() => {
    registerForPushNotificationAsync().then(setDevicePushToken);
  }, []);

  return (
    <View style={styles.container}>
      <Text onPress={() => console.log(devicePushToken)}>{devicePushToken === null ? "nothing yet" : devicePushToken}</Text>
    </View>
  );
}

async function registerForPushNotificationAsync(): Promise<string | null> {
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  if (existingStatus === "denied") {
    return null;
  } else if (existingStatus === "undetermined") {
    const { status } = await Notifications.requestPermissionsAsync();
    if (status !== "granted") {
      return null;
    }
  }
  const token = (await Notifications.getDevicePushTokenAsync()).data as string;

  if (Platform.OS === "android") {
    Notifications.setNotificationChannelAsync("default", {
      name: "default",
      importance: Notifications.AndroidImportance.MAX,
      sound: "notfication_sound.wav",
    });
  }

  return token;
}
```

### Step 9: FCM Authorization Key

In order to be able to send notifications using FCM, we should get hold of an authorization token. To do that do the following or check the [documentation](https://firebase.google.com/docs/cloud-messaging/migrate-v1) if you prefer:

1. Download a service-account file
    - open up your firebase project
    - Click on the gear icon (top left of the screen) > project settings > service accounts
    - click on generate a new key (let it download and save it somewhere)
2. write server side code to generate the token

    - create a new server-side project (nodejs in my case) and write the equivalent of this code:

        ```
        import { google } from "googleapis";

        const MESSAGING_SCOPE = "https://www.googleapis.com/auth/firebase.messaging";
        const SCOPES = [MESSAGING_SCOPE];

        async function getAccessToken() {
          try {
            const key = require("../service-acount.json");
            const jwtClient = new google.auth.JWT(key.client_email, undefined, key.private_key, SCOPES, undefined);
            const tokens = await jwtClient.authorize();
            console.log("access_token: ", tokens.access_token);
          } catch (error) {
            console.log("an error occured");
          }
        }

        getAccessToken();
        ```

        **make sure the downloaded service account file is corrrectly referenced**

    - get the access token from this function and save it somewhere

###Step 10: Send Notifications

#### Android

_pasted from postman, feel free to refactor_

    ```
    var myHeaders = new Headers();
    myHeaders.append("Content-Type", "application/json");
    myHeaders.append("Authorization", "Bearer <access-token-acquired-in previous-step>");

    var raw = JSON.stringify({
      "message": {
        "token": "<NATIVE-DEVICE-PUSH-TOKEN>",
        "android": {
          "priority": "HIGH",
          "notification": {
            "title": "actual title",
            "body": "here is a body",
            "icon": "notification_icon", // base name of icon file configured in app.json
            "color": "#ff0000", // color that overlays icon in notification tray
            "sound": "notification_sound",
            "channelId": "default"
          }
        }
      }
    });

    var requestOptions = {
      method: 'POST',
      headers: myHeaders,
      body: raw,
      redirect: 'follow'
    };

    fetch("https://fcm.googleapis.com/v1/projects/notification-test-520e2/messages:send", requestOptions)
      .then(response => response.text())
      .then(result => console.log(result))
      .catch(error => console.log('error', error));
    ```

#### Ios

In **step 8** we wrote the code that returns the device push token; however, FCM will not accept the token, called APNs token, returned by Ios device.

What we will do is give Firebase the APNs token and it will give us an FCM token. To do that we can do the following request:

```
  var myHeaders = new Headers();
  myHeaders.append("Content-Type", "application/json");
  myHeaders.append("Authorization", "key=<FCM-Server-Key>");

  var raw = JSON.stringify({
    "application": "<IOS-bundle-identifier>",
    "sandbox": false,
    "apns_tokens": [
      "<IOS-Device-APNs-token>"
    ]
  });

  var requestOptions = {
    method: 'POST',
    headers: myHeaders,
    body: raw,
    redirect: 'follow'
  };

  fetch("https://iid.googleapis.com/iid/v1:batchImport", requestOptions)
    .then(response => response.text())
    .then(result => console.log(result))
    .catch(error => console.log('error', error));
```

it will return a response similar to the following:

```
{
  "results": [
      {
          "registration_token": "<FCM-Token-for-ios-device>",
          "apns_token": "f5770663f831290b4eac97e6dfe1b388f100ed895b60775f9a64d7248b883b40",
          "status": "OK"
      }
  ]
}
```

Finally to send the notification to ios:

```
{
	"message":{
	    "token":"<FCM-Token-for-ios-device>",
            "apns": {
                "payload": {
                    "aps": {
                        "alert": {
                            "title": "This is the title",
                            "subtitle": "subtitle goes here",
                            "body": "the bodyyyyy"
                        },
                        "badge": 1,
                        "sound": "notfication_sound.wav"
                    }
                }
            }
    }
}
```

you can customize it to your liking, check out other fields from [docs](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)

If this helped, Don't forget to give it a star 🌟
