## Introduction

We now have the first chunk of our authentication implemented on the
`auth` branch in GitHub (https://github.com/meteor/meteor/tree/auth).

Here are the high-level changes:

1. As part of livedata, method and subscription functions on the
server now have access to the current user ID with `this.userId()`.
This means you can limit what a method does and what data a publish
function sends to the client based on each client's login state.
Since subscriptions are long-lived, Meteor reruns a client's
subscriptions when its user ID changes (eg login or logout).

2. The "accounts" smart package defines a new Meteor.Collection
called "users".  We've written two smart packages that use Facebook or
Google login services (over OAuth2) to manage the current user.

3. The "accounts-ui" smart package provides convenient chrome on the
client for login buttons {{> loginButtons}}.

As an example of these new features, we've added "private" items to
the Todos example, which can be seen at http://auth-todos.meteor.com.
Each user who is logged into todos can now mark items as private,
which no other user can see or modify.  The full diff is here:
https://github.com/meteor/meteor/commit/171816005fa2e263ba54d08d596e5b94dea47b0d

## Getting Started

### Warning
The `auth` branch is a work in progress. The features and API may change at any time.

### Adding Accounts to your app
1. [Get Meteor running from a git checkout](https://github.com/meteor/meteor#slow-start-for-developers). Checkout the `auth` branch.
2. Run `PATH_TO_CHECKOUT/meteor add accounts-ui`
3. Add `{{> loginButtons}}` somewhere in your app. This adds login buttons for whatever services you configure.
4. Add login services -- see below (e.g. `PATH_TO_CHECKOUT/meteor add accounts-google accounts-facebook`)
5. Turn off default mutators or wrap them to check permissions (here's [what we did for todos](https://github.com/meteor/meteor/blob/171816005fa2e263ba54d08d596e5b94dea47b0d/examples/todos/server/access_control.js))
6. You probably want to turn off autopublish (if you want to control which users see which data) 

### Updates to the API
#### Basics
- [Client/Server] `Meteor.users` is a collection of all users. By default the current user's public fields (eg "emails" and "name") are published to all clients. If autopublish is enabled all public fields of all users are published. You can choose to publish any additional fields -- overlapping subscriptions should work fine.
- [Client/Server] Within methods/subscriptions -- `this.userId()` returns the current user ID
- [Client] `Meteor.user()` is a reactive function returning the current logged in user document, or null if the user is logged out
- [Client] A global Handlebars helper named `currentUser` (e.g. `{{#if currentUser}}Make private{{/if}}`)

#### Disabling full write access
By default, clients are given full write access to all collections. To turn this behavior off, remove the `insecure` package

#### Restricting writes
[Server] `collection.allow(options)` Restricts default mutator methods on this collection. Once this is called, all write methods on this collection are secured. Multiple calls add more restrictions. Works with or without the `insecure` package.

Options:
- `insert` (Function(userId, doc)) - Return true to allow user to insert document
- `update` (Function(userId, docs, fields, modifier)) - Return true to allow user to update documents
 - `fields` - Array of fields to be modified
 - `modifier` - Original mongo [modifier operation](http://www.mongodb.org/display/DOCS/Updating#Updating-ModifierOperations)
- `remove` (Function(userId, docs)) - Return true to allow user to remove documents
- `fetch` (Array) - Fields to be fetched for update and remove restrictions (if not passed, all fields will be fetched)


#### If you aren't using accounts-ui
- [Client] `Meteor.loginWithFacebook(callback)`
- [Client] `Meteor.loginWithGoogle(callback)`
- [Client] `Meteor.logout(callback)`

#### Configuring login services (see section below)
- [Client/Server] `Meteor.accounts.facebook.config(appId, appUrl)`
- [Client/Server] `Meteor.accounts.google.config(clientId, appUrl)`
- [Server] `Meteor.accounts.facebook.setSecret(appSecret)`
- [Server] `Meteor.accounts.google.setSecret(clientSecret)`

## Integrating with Login Services

Note your application's deployed URL. We'll refer to that as `APP_URL`

### Facebook
1. Register your app on Facebook, noting your app ID and secret, which we will refer to as to as `APP_ID` and `APP_SECRET` (More details on this below.)
2. Run `PATH_TO_CHECKOUT/meteor add accounts-facebook`
3. Add the following line to a file visible to both client and server, e.g. `accounts/providers.js`:
```
Meteor.accounts.facebook.config(APP_ID, APP_URL);
```

4. Add the following line to a file visible only to the server, e.g. `accounts/server/provider_secrets.js`:
```
Meteor.accounts.facebook.setSecret(APP_SECRET);
```

#### Registering your app on Facebook (step 1 above)
1. Go to https://developers.facebook.com/apps
2. Click "Create new App"
3. You only need to set a name
4. Under "Select how your app integrates with Facebook", expand "Website with Facebook Login". Make sure to set the app URL (If you're running locally, "http://localhost:3000" works)


### Google
1. Get an Google client ID and secret, which we will refer to as to as `CLIENT_ID` and `CLIENT_SECRET` (More details on this below.) Make sure to allow `APP_URL/_oauth/google?close` as a authorized redirect URI
2. Run `PATH_TO_CHECKOUT/meteor add accounts-google`
3. Add the following line to a file visible to both client and server, e.g. `accounts/providers.js`:
```
Meteor.accounts.google.config(CLIENT_ID, APP_URL);
```

4. Add the following line to a file visible only to the server, e.g. `accounts/server/provider_secrets.js`:
```
Meteor.accounts.google.setSecret(CLIENT_SECRET);
```


#### Getting a Google client ID (step 1 above)
1. Go to https://code.google.com/apis/console/
2. Open the "API Access" tab
3. Click on "Create another client ID"
4. Click on "More options"
5. Authorized Redirect URIs: Should contain APP_URL/_oauth/google?close
6. Authorized JavaScript origins: APP_URL
7. Click on 'Create Client ID'