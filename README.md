# Passkeys Hands on Lab - Twilio SIGNAL 2025

## Pre-requisites

Follow the [IT checklist](it-checklist.md) to make sure you have everything you'll need for building.

## Step 1: Deploy the starter application from the Twilio CodeExchange 

Head over to the [Twilio Code Exchange and deploy this sample project as a starting point](https://www.twilio.com/code-exchange/verify-passkeys). This project will provide a few things:

* A live URL you can use as your [relying party](https://webauthn.wtf/how-it-works/relying-party) for testing passkeys
* Hosted functions to register and authenticate with passkeys

After you deploy your project, click the option to Edit your customizations directly in code. This will take you to the functions editor in the Twilio Console.

<img width="600" alt="image" src="https://github.com/user-attachments/assets/8b202f50-d26c-45e3-947d-850ff4884a27" />

## Step 2: Use the Twilio Passkey JavaScript SDK to create and manage passkeys in the browser

Next you will create a frontend for your application to interact with the [built-in web authentication browser APIs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API). To do this, you will integrate The Twilio Passkey JavaScript SDK which will allow you to easily create and fetch passkeys, as well as interact with the backend API you just created for authentication.

Create a new asset called **index.html** and paste the following starter code:

```javascript
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="x-ua-compatible" content="ie=edge" />
    <title>Passkeys Demo</title>

    <link
      rel="icon"
      href="https://twilio-labs.github.io/function-templates/static/v1/favicon.ico"
    />
    <link
      rel="stylesheet"
      href="https://twilio-labs.github.io/function-templates/static/v1/ce-paste-theme.css"
    />
    <link
      rel="stylesheet"
      href="https://raw.githubusercontent.com/robinske/passkeys-workshop/refs/heads/main/assets/styles.css"
    />
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/intl-tel-input@19.5.3/build/css/intlTelInput.css"
    />
    <script src="https://cdn.jsdelivr.net/npm/intl-tel-input@19.5.3/build/js/intlTelInput.min.js"></script>
  </head>
  <body>
    <div id="container">
      <h1 class="title">Sign up or sign in</h1>
      <div class="input_container">
        <div class="input_component">
          <input
            type="tel"
            name="username_input"
            id="usr_input"
            oninput="checkAvalibility()"
          />
        </div>
        <button
          type="button"
          class="btn continue_btn"
          id="continue"
          onclick="login()"
          disabled
        >
          Continue
        </button>
        <span class="separator">&#8213; or &#8213;</span>
        <button type="button" class="btn passkey_btn" onclick="signIn()">
          Sign in with passkey
        </button>
      </div>
    </div>
    <div id="modal" class="invisible">
      <h1 class="title">
        Sign-in with your face,<br />
        fingerprint or PIN
      </h1>
      <p>
        Harness your device capabilities for a fast<br />
        passkey login with maximun security.
      </p>
      <a>Learn more &rarr;</a>
      <div class="input_container modal_input">
        <button class="btn continue_btn enable" onclick="signUp()">
          Continue
        </button>
        <a class="skip_btn">Skip</a>
      </div>
    </div>
    <div id="app" class="invisible">
      <h1 class="title" id="welcome"></h1>
      <div class="input_container">
        <button class="btn continue_btn enable" onclick="logOut()">
          Log out
        </button>
      </div>
    </div>
  </body>
  <script>
    let sessionUsername = sessionStorage.getItem("session");

    const loadApp = (username) => {
      document.getElementById("welcome").innerHTML = `Welcome ${username}`;
      document.getElementById("modal").classList.add("invisible");
      document.getElementById("container").classList.add("invisible");
      document.getElementById("app").classList.remove("invisible");
    };

    if (sessionUsername) {
      loadApp(sessionUsername);
    }

    const usernameElement = document.getElementById("usr_input");
    const errorElement = document.getElementById("error");
    const continueButton = document.getElementById("continue");
    const iti = window.intlTelInput(usernameElement, {
      initialCountry: "us",
      showSelectedDialCode: true,
      utilsScript:
        "https://cdn.jsdelivr.net/npm/intl-tel-input@19.5.3/build/js/utils.js",
    });

    const checkAvalibility = () => {
      const username = usernameElement.value;
      if (username) {
        continueButton.classList.add("enable");
        continueButton.disabled = false;
      } else {
        continueButton.classList.remove("enable");
        continueButton.disabled = true;
      }
    };

    const login = () => {
      const authenticationCard = document.getElementById("container");
      const passkeyCard = document.getElementById("modal");
      authenticationCard.classList.add("invisible");
      passkeyCard.classList.remove("invisible");
    };

    const signUp = async () => {
      const username = iti.getNumber(
        window.intlTelInputUtils.numberFormat.E164
      );

      try {
        //here will be the Sign Up logic
      } catch (error) {
        console.log(error);
      }
    };

    const signIn = async () => {
      try {
        // here will be the Sign In logic
      } catch (error) {
        console.log(err);
      }
    };

    const logOut = () => {
      sessionStorage.removeItem("session");
      document.getElementById("app").classList.add("invisible");
      document.getElementById("container").classList.remove("invisible");
    };
  </script>
</html>
```

This provides placeholders for sign up and sign in functions that we will fill in next.

Now it is time to add the SDK. For this tutorial you will load it from the [CDN](https://github.com/twilio/twilio-verify-passkeys-web/tree/0.0.2?tab=readme-ov-file#using-it-from-cdn) in the **index.html** file. Add the following script tag inside the **`<head>`** element after the **`<title>`**: 

```javascript
<script src="https://unpkg.com/@twilio/twilio-verify-passkeys-web@0.0.2/dist/twilio-verify-passkeys.iife.js"></script>
```

After that you can instantiate the SDK. Add the following line inside the script block before the **`const signUp`** function.

```javascript
const twilioPasskeys = new TwilioPasskeys();
```

To create a passkey at sign up, add the following code inside the **`try`** block of the signUp function.

```javascript
const response = await fetch(`./registration/start`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ username }),
});

const responseJSON = await response.json();

let credential = await twilioPasskeys.create(JSON.stringify(responseJSON));
if (credential.Error != null) throw new Error(credential.Error);

const verificationResponse = await fetch(`./registration/verification`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    id: credential.Success.id,
    rawId: credential.Success.rawId,
    authenticatorAttachment: credential.Success.authenticatorAttachment,
    type: credential.Success.type,
    response: {
      attestationObject: credential.Success.attestationObject,
      clientDataJSON: credential.Success.clientDataJSON,
      transports: credential.Success.transports,
    },
  }),
});

const { status } = await verificationResponse.json();
if (status === "verified") {
  sessionStorage.setItem("session", username);
  loadApp(username);
}
```

Do the same for the signIn function. This will fetch a passkey from the user's password manager and verify it with the backend:	

```javascript
const response = await fetch(`./authentication/start`);
const responseJSON = await response.json();

let { Success, Error } = await twilioPasskeys.authenticate(
  JSON.stringify(responseJSON)
);

const authentication = await fetch(`./authentication/verification`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    id: Success.id,
    rawId: Success.rawId,
    authenticatorAttachment: Success.authenticatorAttachment,
    type: Success.type,
    response: {
      authenticatorData: Success.authenticatorData,
      clientDataJSON: Success.clientDataJSON,
      signature: Success.signature,
      userHandle: Success.userHandle,
    },
  }),
});

const { status, identity } = await authentication.json();
if (status === "approved") {
  sessionStorage.setItem("session", identity);
  loadApp(identity);
} else {
  console.log(status);
}
```

## Step 3: Test the application

Make sure your **index.html** asset is set to Public then save and deploy all of your functions and assets. Copy the URL and test your app by registering a new passkey!
