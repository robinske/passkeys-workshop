# **Passkeys Hands on Lab \- Twilio SIGNAL 2025**

## **Pre-requisites**

Follow the [IT checklist](https://github.com/robinske/passkeys-workshop/blob/main/it-checklist.md) to make sure you have everything you'll need for building.

## **Step 1: Deploy the starter application from the Twilio CodeExchange**

Head over to the [Twilio Code Exchange and deploy this sample project as a starting point](https://www.twilio.com/code-exchange/verify-passkeys). Use the default **Api url** and leave **Android app keys** blank \- you won't need them for this workshop.

This Code Exchange project will provide a few things:

* A live URL you can use as your [relying party](https://webauthn.wtf/how-it-works/relying-party) for testing passkeys  
* Hosted functions to register and authenticate with passkeys, managing the backend interactions with the Verify Passkeys API

After you deploy your project, click the option to **"Edit your customizations directly in code"**. This will take you to the functions editor in the Twilio Console.

<img width="600" alt="image" src="https://github.com/user-attachments/assets/8b202f50-d26c-45e3-947d-850ff4884a27" />

## **Step 2: Use the Twilio Passkeys JavaScript SDK to create and manage passkeys in the browser**

Next, create a frontend for your application to interact with the [built-in web authentication browser APIs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API). To do this, you will integrate the Twilio Passkey JavaScript SDK to easily create and fetch passkeys, as well as interact with the backend API you just created for authentication.

> [!TIP]
> Twilio Functions does not have a built-in code formatter. You can use [https://freetools.textmagic.com/source-code-formatter](https://freetools.textmagic.com/source-code-formatter) to periodically format your code as you're developing in the console.

Replace the code in "**index.html"** with the following starter code:

```javascript
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="x-ua-compatible" content="ie=edge" />
    <title>Passkeys Demo</title>

    <!-- TODO: add script tag for SDK here -->

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
      href="https://cdn.jsdelivr.net/npm/intl-tel-input@25.3.1/build/css/intlTelInput.css"
    />
    <link
      rel="stylesheet"
      href="https://robinske.github.io/passkeys-workshop/styles.css"
    />
    <script src="https://cdn.jsdelivr.net/npm/intl-tel-input@25.3.1/build/js/intlTelInput.min.js"></script>
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
            oninput="toggleContinueButton()"
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
      loadUtils: () =>
        import(
          "https://cdn.jsdelivr.net/npm/intl-tel-input@25.3.1/build/js/utils.js"
        ),
    });

    const toggleContinueButton = () => {
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

    // TODO - instantiate the TwilioPasskeys SDK here

    const signUp = async () => {
      const username = iti.getNumber(
        window.intlTelInput.utils.numberFormat.E164
      );

      try {
        // TODO - Add sign up logic
      } catch (error) {
        console.error(error);
      }
    };

    const signIn = async () => {
      try {
        // TODO - Add sign in logic
      } catch (error) {
        console.error(err);
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

This provides placeholders for sign up and sign in functions that we will create next.

Next, load the Passkeys Javascript SDK from the [CDN](https://github.com/twilio/twilio-verify-passkeys-web/tree/0.0.2?tab=readme-ov-file#using-it-from-cdn) in the index.html file. Add the following script tag inside the `<head>` element after the `<title>`:

```javascript
<script src="https://unpkg.com/@twilio/twilio-verify-passkeys-web@0.0.2/dist/twilio-verify-passkeys.iife.js"></script>
```

After that you can instantiate the SDK. Add the following line inside the script block before the `const signUp` function.

```javascript
const twilioPasskeys = new TwilioPasskeys();
```

To create a passkey at sign up, add the following code inside the `try` block of the signUp function, replacing the `// TODO - Sign Up logic:` 

```javascript
        const response = await fetch(`./registration/start`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ username }),
        });
        const publicKey = await response.json();
        let credential = await twilioPasskeys.create(JSON.stringify(publicKey));
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
          console.log("Passkey created and verified on the server, saving session...");
          sessionStorage.setItem("session", username);
          loadApp(username);
        }
```
## **Step 3: Test out creating a passkey**

Make sure your **"index.html"** asset is set to Public then save and deploy all of your functions and assets. Copy the full URL and test your app by registering a new passkey.  
![image](https://github.com/user-attachments/assets/7b5bddfb-924f-4af0-aa75-bb14ae095e73)


Enter your phone number and follow the prompts to register a passkey. Here I'm registering a passkey with the Google Chrome Password Manager, but you can use the passkey storage service of your choice (e.g. iCloud Keychain, 1Password, and more).

![image](https://github.com/user-attachments/assets/893b17ee-f3ee-465d-8a38-5c7ede49bfe0)


## **Step 4: Add login support**

Next, add support for logging in. This flow will fetch the passkey from the user's password manager (using the JavaScript SDK) and validate its signature with the server (using the Twilio Verify API).

Replace the `TODO - sign in logic` with the following code:

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
          console.log("Passkey validated on the server, logging in...");
          sessionStorage.setItem("session", identity);
          loadApp(identity);
        } else {
          console.log(status);
        }
```

## **Step 5: Test signing in with a passkey**

Save and re-deploy all of your functions and assets. Copy the URL and test your app by signing in with the passkey you created.
