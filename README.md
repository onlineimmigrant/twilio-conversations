# Conversations Demo Web Application Overview

SDK version of this demo app: ![](https://img.shields.io/badge/SDK%20version-2.0.0-blue.svg)

The latest available SDK version of this demo app: ![](https://img.shields.io/badge/SDK%20version-2.0.0-green.svg)

## Getting Started

Welcome to the Conversations Demo Web application. This application demonstrates a basic Conversations client application with the ability to create and join conversations, add other participants into the conversations and exchange messages.

You can try out one of our 1-click deploys to test the app out prior to jumping to [Next Steps](#next-steps):

## Deploy

### Test out on Github Codespaces

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/twilio/twilio-conversations-demo-react)

Note: This deployment requires a [token service url](#generating-access-tokens).

### Deploy to Vercel

Automatically clone this repo and deploy it through Vercel. 

Note: This deployment requires a [token service url](#generating-access-tokens). Vercel will ask for the `REACT_APP_ACCESS_TOKEN_SERVICE_URL` env variable. 

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Ftwilio%2Ftwilio-conversations-demo-react%2F&env=REACT_APP_ACCESS_TOKEN_SERVICE_URL&envDescription=A%20link%20to%20your%20access%20token%20server.%20Use%20the%20Twilio%20Functions%20example%20from%20the%20readme%20for%20quick%20testing.&envLink=https%3A%2F%2Fgithub.com%2Ftwilio%2Ftwilio-conversations-demo-react%2F%23generating-access-tokens&project-name=twilio-conversations&repository-name=twilio-conversations)

## Next Steps

What you'll need to get started:

- A [fork](https://github.com/twilio/twilio-conversations-demo-react/fork) of this repo to work with.
- A way to [generate Conversations access tokens](#generating-access-tokens).
- Optional: A Firebase project to [set up push notifications](#setting-up-push-notifications).

## Generating Access Tokens

Client apps need [access tokens](https://www.twilio.com/docs/conversations/create-tokens) to authenticate and connect to the Conversations service as a user. These tokens should be generated by your backend server using your private Twilio API Keys. If you already have a service that does this, skip to [setting the token service URL](#set-the-token-service-url). 

For testing purposes, you can quickly set up a Twilio Serverless Functions to generate access tokens. Note that this is not a production ready implementation.

### Generating Access Tokens with Twilio Functions

1. Create a [Twilio Functions Service from the console](https://www.twilio.com/console/functions/overview) and add a new function using the **Add+** button.
2. Set the function path name to `/token-service`
3. Set the function visibility to `Public`. 
4. Insert the following code:

```javascript
// If you do not want to pay for other people using your Twilio service for their benefit,
// generate user and password pairs different from what is presented here
let users = {
    user00: "", !!! SET NON-EMPTY PASSWORD AND REMOVE THIS NOTE, THIS GENERATOR WILL NOT WORK WITH EMPTY PASSWORD !!!
    user01: ""  !!! SET NON-EMPTY PASSWORD AND REMOVE THIS NOTE, THIS GENERATOR WILL NOT WORK WITH EMPTY PASSWORD !!!
};

let response = new Twilio.Response();
let headers = {
    'Access-Control-Allow-Origin': '*',
  };

exports.handler = function(context, event, callback) {
    response.setHeaders(headers);
    if (!event.identity || !event.password) {
        response.setStatusCode(401);
        response.setBody("No credentials");
        callback(null, response);
        return;
    }

    if (users[event.identity] != event.password) {
        response.setStatusCode(401);
        response.setBody("Wrong credentials");
        callback(null, response);
        return;
    }
    
    let AccessToken = Twilio.jwt.AccessToken;
    let token = new AccessToken(
      context.ACCOUNT_SID,
      context.TWILIO_API_KEY_SID,
      context.TWILIO_API_KEY_SECRET, {
        identity: event.identity,
        ttl: 3600
      });

    let grant = new AccessToken.ChatGrant({ serviceSid: context.SERVICE_SID });
    if(context.PUSH_CREDENTIAL_SID) {
      // Optional: without it, no push notifications will be sent
      grant.pushCredentialSid = context.PUSH_CREDENTIAL_SID; 
    }
    token.addGrant(grant);
    response.setStatusCode(200);
    response.setBody(token.toJwt());

    callback(null, response);
};
```

5. Save the function.
6. Open the **Environment Variables** tab from the **Settings** section and: 
    - Check the **"Add my Twilio Credentials (ACCOUNT_SID) and (AUTH_TOKEN) to ENV"** box, so that you get `ACCOUNT_SID` automatically.
    - Add SERVICE_SID
        - Open [Conversations Services](https://www.twilio.com/console/conversations/services)
        - Copy the `SID` for `Default Conversations Service`, or the service you want to set up.
    - Add `TWILIO_API_KEY_SID` and `TWILIO_API_KEY_SECRET`. Create API Keys [in the console](https://www.twilio.com/console/project/api-keys).
    - Optionally add `PUSH_CREDENTIAL_SID`, for more info see [Setting up Push Notifications](#setting-up-push-notifications)
7. **Copy URL** from the "kebab" three dot menu next to it and and use it as `REACT_APP_ACCESS_TOKEN_SERVICE_URL` .env variable below.
8. Click **Deploy All**.

### Set the Token Service URL

If you don't have your own `.env`, rename this repo's `.env.example` file to `.env`. Set the value of `REACT_APP_ACCESS_TOKEN_SERVICE_URL` to point to a valid Access Token server. If you used Twilio Functions for generating tokens, get the value from **Copy URL** in step 7 above.  

```
REACT_APP_ACCESS_TOKEN_SERVICE_URL=http://example.com/token-service/
```

NOTE: No need for quotes around the URL, they will be added automatically.

This demo app expects your access token server to provide a valid token for valid credentials by URL:

 ```
$REACT_APP_ACCESS_TOKEN_SERVICE_URL?identity=<USER_PROVIDED_USERNAME>&password=<USER_PROVIDED_PASSWORD>
 ```
And return HTTP 401 in case of invalid credentials.

## Setting up Push Notifications

This demo app uses Firebase for processing notifications. This setup is optional. Note: Support may be limited for some browswers. 

### Set up Firebase

1. Create a [Firebase project](https://firebase.google.com)
2. Go to the [Project Settings](https://console.firebase.google.com/project/_/settings/general/)
3. Got to the **Cloud Messaging** and enable **Cloud Messaging API (Legacy)** through the "kebab" menu besides it.
4. Note or copy the Server Key token for creating push credentials. 

### Create Push Credential

Create a push credential to add a push grant to our access token. 
1. Go to the [Credentials](twilio.com/console/project/credentials/push-credentials) section of the console.
2. Create a new FCM Push Credential and set the Firebase Cloud Message Server Key Token as the `FCM Secret`.
3. Note or copy the `CREDENTIAL SID` to set as `PUSH_CREDENTIAL_SID` env variable in your token creation Function. 

### Create Firebase App set config

From the Firebase [Project Settings](https://console.firebase.google.com/project/_/settings/general/) **General** tab, add a web app to get the `firebaseConfig`, it should look like this:

```javascript
var firebaseConfig = {
  apiKey: "sample__key12345678901234567890",
  authDomain: "convo-demo-app-internal.firebaseapp.com",
  projectId: "convo-demo-app-internal",
  storageBucket: "convo-demo-app-internal.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:1234abcd",
  measurementId: "EXAMPLE_ID"
};
```

Note: Firebase API Keys aren't like Twilio API keys and don't need to be kept secret. 

Replace this project's `firebase-config.example` in the `public` folder with a `firebase-config.js` containing your specific config.  

### Enable Push Notification in Conversations Service

Select your [conversations service](https://www.twilio.com/console/conversations/services), navigate to the **Push Notifications** section, and check the **Push notifications enabled** boxes for the push notifications you want. 

## Build & Run

### Deploy on Github Codespaces

- Click [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/twilio/twilio-conversations-demo-react)
- Wait for the pop-up message to let you know that the port forwarding is done. Then, click "Open in Browser".
- If the pop-up message isn't displayed, you can always open "PORTS" tab and click on "Open in Browser" button manually.

### Run Application Locally

- Run `yarn` to fetch project dependencies.
- Run `yarn build` to fetch Twilio SDK files and build the application.
- Run `yarn start` to run the application locally.

### Run Application Inside Docker

- Run `docker compose up --build` to build and locally run the application inside a Docker container.

## License

MIT

## Projects
[https://metexam.co.uk](https://metexam.co.uk)




