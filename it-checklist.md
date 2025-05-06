# IT Testing Checklist

- [ ] Can you log in to your [Twilio account/console?](https://twilio.com/console)
- [ ] Are you working in a separate project or subaccount to prevent overriding production settings? Instructions to: [View and Create new Twilio Subaccounts](https://support.twilio.com/hc/en-us/articles/360011348693-View-and-Create-New-Twilio-Subaccounts)  
- [ ] Do you have your [API Credentials](https://www.twilio.com/docs/iam/credentials/api)? We will be using our Account SIDs and Auth Tokens
- [ ] Send the Account SID you plan to use for the day of the build event to [tgu@twilio.com](mailto:tgu@twilio.com). Account SIDs are formatted like: `ACxxxxxxxxxxx...` and can be found [in the Twilio Console](https://console.twilio.com/us1/account/manage-account/general-settings)

**Once you provided your Account SID:** 

- [ ] Check that you have access to Verify Passkeys on your Account (cURL). Access will usually be granted in 24-48 hrs after receiving your Account SID. You can run a quick cURL command below using your Twilio Account SID and Auth Token to check if access has been granted.

```curl
# Replace $TWILIO_ACCOUNT_SID with your Account Sid
# Replace $TWILIO_AUTH_TOKEN with your Auth Token

curl --location 'https://comms.twilio.com/preview/Factors' \
--header 'Content-Type: application/json' \
--data '{
  "friendly_name":"Test Factor",
  "to":{
     "user_identifier":"test-user"
  },
  "content":{
     "relying_party":{
        "id":"rp-123",
        "name":"rp-123",
        "origins":[
           "authenticator-123"
        ]
     },
     "user":{
        "display_name":"User 123"
     }
  }
}' \
-u $TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN
```

Prerequisites for using Verify Passkeys Web SDK

- [ ] Tech Stack: The Web SDK is built with Vanilla Javascript and uses JSDoc for annotations, so only Javascript is supported for building sample app.  
- [ ] You can create a [Twilio Function](https://www.twilio.com/docs/runtime/quickstart/serverless-functions-send-sms)
  - This will be used to create the sample app and the sample backend during our session
- [ ] If you would like to connect to your own backend, please have ngrok configured on your machine if it is running in your local machine
  - ngrok instructions [here](https://ngrok.com/docs/guides/getting-started/)  
- [ ] Will you be running your code from your local machine? If so:   
  - Download the Twilio CLI and set it up on your machine → [Installation instructions](https://www.twilio.com/docs/twilio-cli/quickstart)  
