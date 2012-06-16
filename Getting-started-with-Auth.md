# Introduction

Copy email contents

# Getting Started

## Adding Accounts to your app
Here's how we did it for the Todos sample app: XXX

1. Run `meteor add accounts-ui`
2. Add {{> loginButtons}} somewhere in your app
3. Add appropriate login services -- see below (e.g. `PATH_TO_CHECKOUT/meteor add accounts-google accounts-facebook`)
4. Turn off default mutators or wrap them to check permissions (see the Todos example above)
5. You probably want to turn off autopublish (if you want to control which users see which data) 

## Updates to the API
- [Client/Server] Meteor.users is a collection of all users. By default the current user's public fields (eg "emails" and "name") are published to all clients. If autopublish is enabled all public fields of all users are published. You can choose to publish any additional fields -- overlapping subscriptions should work fine.
- [Client/Server] Within methods/subscriptions -- this.userId() returns the current e.g. privateTo in subs; e.g. check permissions in method calls (probably in server-only code)
- [Client] Meteor.user() is a reactive function returning the current logged in user document
- [Client] 
- {{#if user}}Make private{{/if user}}

In addition, if you prefer not to use the login-buttons package, you can build your own login buttons using the following functions, 
- `Meteor.loginWithFacebook(callback)`
- `Meteor.loginWithGoogle()`
- `Meteor.logout()`

And the provider setup functions XXX



Some description of the internal APIs?
The oauth package API
Package list: oauth2, accounts, accounts-google, accounts-facebook, login-buttons
make oauth2 internal
rename login-buttons to accounts-ui



# Integrating with Login Services

Note your application's deployed URL. We'll refer to that as `APP_URL`

## Facebook
1. Register your app on Facebook, noting your app id and secret, which we will refer to as to as `APP_ID` and `APP_SECRET` (More details on this below.)
2. Run `PATH_TO_CHECKOUT/meteor add accounts-facebook`
3. Add the following line to a file visible to both client and server, e.g. `accounts/providers.js`:
```
Meteor.accounts.facebook.setup(APP_ID, APP_URL)
```

4. Add the following line to a file visible only to the server, e.g. `accounts/server/provider_secrets.js`:
```
Meteor.accounts.facebook.setSecret(APP_SECRET)
```

### Registering your app on Facebook (step 1 above)
1. Go to https://developers.facebook.com/apps
2. Click "Create new App"
3. You only need to set a name
4. Under "Select how your app integrates with Facebook", expand "Website with Facebook Login". Make sure to set the app URL (If you're running locally, "http://localhost:3000" works)
 

## Google
1. Get an Google client ID, your client secret, which we will refer to as to as `CLIENT_ID` and `CLIENT_SECRET` (More details on this below.) Make sure to allow `APP_URL/_oauth/google?close` as a authorized redirect URI
2. Run `PATH_TO_CHECKOUT/meteor add accounts-google`
3. Add the following line to a file visible to both client and server, e.g. `accounts/providers.js`:
```
Meteor.accounts.google.setup(CLIENT_ID, APP_URL)
```

4. Add the following line to a file visible only to the server, e.g. `accounts/server/provider_secrets.js`:
```
Meteor.accounts.google.setSecret(CLIENT_SECRET)
```


### Getting a Google client ID (step 1 above)
1. Go to https://code.google.com/apis/console/
2. Open the "API Access" tab
3. Click on "Create another client ID"
4. Click on "More options"
5. Authorized Redirect URIs: If your app is on 'http://localhost:3000' this should contain 'http://localhost:3000/_oauth/google?close'
6. Authorized JavaScript origins: 'http://localhost:3000'
7. Click on 'Create Client ID'


