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

2. The "accounts" smart package defines a new Meteor.Collection called
"users".  We've written several packages that manage this collection
using both third party services such Facebook and Google (via OAuth2)
and old fashioned usernames and passwords.

3. The "accounts-ui" smart package provides convenient chrome on the
client for a login form {{> loginButtons}}.

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
4. Add login services -- see below (e.g. `PATH_TO_CHECKOUT/meteor add accounts-google accounts-facebook accounts-passwords`)
5. Restrict writes (here's [what we did for todos](https://github.com/meteor/meteor/blob/171816005fa2e263ba54d08d596e5b94dea47b0d/examples/todos/server/access_control.js))
6. You probably want to turn off autopublish (if you want to control which users see which data) 

### Updates to the API
#### Basics
- [Client/Server] `Meteor.users` is a collection of all users. By default the current user's public fields (eg "emails" and "name") are published to all clients. If autopublish is enabled all public fields of all users are published. You can choose to publish any additional fields -- overlapping subscriptions should work fine.
- [Client/Server] Within methods/subscriptions -- `this.userId()` returns the current user ID
- [Client] `Meteor.user()` is a reactive function returning:
 - user document if the user is logged in and the user document data is fully loaded on the client
 - `null` if the user is logged out
 - `{_id: (user id), loading: true}` if the user is logged in but we are still waiting for the subscription to load on the client 
- [Client] A global Handlebars helper named `currentUser` (e.g. `{{#if currentUser}}Make private{{/if}}`)

#### Configuration
[Client/Server] `Meteor.accounts.config(options)` - Global configuration of the accounts system. Affects both the low-level API and the appearance of `accounts-ui`. 

NOTE: We are fairly confident that this API will change.

Options:
- `requireEmail` (Boolean) - Require users created to have an email. This also effects the login and signup forms in `accounts-ui`
- `requireUsername` (Boolean) - Require users created to have a username. This also effects the login and signup forms in `accounts-ui`
- `validateEmails` (Boolean) - Send validation emails to users created via `accounts-password` (as opposed to users created via OAuth, whose email is assumed to be validated).

#### Disabling full write access
By default, clients are given full write access to all collections. To turn this behavior off, remove the `insecure` package

#### Restricting writes
[Server] `collection.allow(options)` Restricts default write methods on this collection. Once this is called, all write methods on this collection are secured. Multiple calls add more restrictions. Works with or without the `insecure` package.

Options:
- `insert` (Function(userId, doc)) - Return true to allow user to insert document
- `update` (Function(userId, docs, fields, modifier)) - Return true to allow user to update documents
 - `fields` - Array of fields to be modified
 - `modifier` - Original mongo [modifier operation](http://www.mongodb.org/display/DOCS/Updating#Updating-ModifierOperations)
- `remove` (Function(userId, docs)) - Return true to allow user to remove documents
- `fetch` (Array) - Fields to be fetched for update and remove restrictions (if not passed, all fields will be fetched)

#### Email templates

[Server] `Meteor.accounts.emailTemplates` - An object that can be modified to customize the emails that are sent.

#### Low-level API
If you're not using `accounts-ui`, use these functions to implement your own login flow. You'll also have to handle the special URLs sent in emails by showing dialogs for email validation, reset password and account enrollment. You can also use `accounts-ui` without `{{> loginButtons}}` if you just want to get the dialogs.

- [Client] `Meteor.loginWithFacebook()`
- [Client] `Meteor.loginWithGoogle()`
- [Client] `Meteor.loginWithWeibo()`
- [Client] `Meteor.loginWithTwitter()`
- [Client] `Meteor.loginWithPassword(user, password, callback)`
 - `user` argument is either `{username: 'username'}`, `{email: 'email@address'}`, or a string that might be username or email.
 - `password`: the plaintext password. The password is _not_ sent unencrypted, though.
 - `callback`: Function(error|null)
- [Client] `Meteor.logout()`
- [Client] `Meteor.createUser(options, extra, callback)` - Creates a user and logs in as that user
 - `options` a hash containing: `username` and/or `email`, `password`
 - `extra`: extra fields for the user object (eg `name`, etc).
 - `callback`: Function(error|null)
- [Server] `Meteor.createUser(options, extra)` - Creates a user and sends that user an email with a link to choose their initial password and complete their account enrollment
 - `options` a hash containing: `email` (mandatory), `username` (optional)
 - `extra`: extra fields for the user object (eg `name`, etc).
- [Client] `Meteor.changePassword(oldPassword, newPassword, callback)`
 - `callback`: Function(error|null)
 - Must be logged in to call this. Changes the currently logged in user.
- [Client] `Meteor.enrollAccount(token, password, callback)` - Completes the account enrollment process that began with a server-side call to `Meteor.createUser`. Also validates this user's email address.
 - `token`: unique string contained in the email sent by `Meteor.createUser`
 - `callback`: Function(error|null)
- [Client] `Meteor.forgotPassword(options, callback)` - Requests that a reset password link be sent to a user
 - `options`: object containing an `email` field
 - `callback`: Function(error|null)
- [Client] `Meteor.resetPassword(token, newPassword, callback)` - Resets a user's password
 - `token`: unique string contained in the email sent to the user by `Meteor.forgotPassword`
 - `callback`: Function(error|null)
- [Client] `Meteor.validateEmail(token, callback)` - Validate a user's email
 - `token`: unique string contained in the email sent to the user by a client-side call to `Meteor.createUser` (in case `validateEmails` was set to true in the call to `Meteor.accounts.config`)
 - `callback`: Function(error|null)


#### Configuring login services (see section below)
- [Client/Server] `Meteor.accounts.facebook.config(appId, appUrl, options)`
- [Client/Server] `Meteor.accounts.google.config(clientId, appUrl, options)`
- [Server] `Meteor.accounts.facebook.setSecret(appSecret)`
- [Server] `Meteor.accounts.google.setSecret(clientSecret)`

Options:
- scope: a list of permissions to request when logging in. For example, on Facebook: `scope: ["user_birthday", "user_checkins"]`. On Google: `scope: ["https://www.googleapis.com/auth/calendar"]`


#### Controlling new user creation

- [Server] `Meteor.accounts.validateNewUser(validator)`
 - `validator`: Function(proposedUser). returns true to allow user creation, false to deny.
 - Can be called multiple times. All validators must pass for the user to be created.
- [Server] `Meteor.accounts.onCreateUser(Function(options, extra, user))`
 - `options`: Basic parameters for user creation. eg `username`, `email`.
 - `extra`: extra fields proposed for the new user. straight from the client. eg `name`.
 - `user`: A pre-processed user object with transformed options.
 - return value of function: a proposed user object, with all fields filled out. This can be based on the user object passed in, or constructed totally differently. throw an error to abort user creation.
 - This can only be set once. If it is not set, the default implementation simply copies the 'extra' fields into the user object.



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


### Weibo
1. Get an Weibo client ID and secret, which we will refer to as to as `CLIENT_ID` and `CLIENT_SECRET` (More details on this below.) Make sure to allow `APP_URL/_oauth/weibo?close` as a authorized redirect URI
2. Run `PATH_TO_CHECKOUT/meteor add accounts-weibo`
3. Add the following line to a file visible to both client and server, e.g. `accounts/providers.js`:
```
Meteor.accounts.weibo.config(CLIENT_ID, APP_URL);
```

4. Add the following line to a file visible only to the server, e.g. `accounts/server/provider_secrets.js`:
```
Meteor.accounts.weibo.setSecret(CLIENT_SECRET);
```


#### Getting a Weibo client ID (step 1 above)
1. Go to http://open.weibo.com/development
2. Click the "创建应用" button
3. Follow the guide to create an application
4. Authorized Redirect URIs: Should contain APP_URL/_oauth/weibo?close
5. Their `App Key` is `CLIENT_ID` we need, and `App Secret` is `CLIENT_SECRET`

### Twitter
1. Get a Twitter consumer key and secret, which we will refer to as to as `CONSUMER_KEY` and `CONSUMER_SECRET` (More details on this below.) Make sure to allow `APP_URL/_oauth/twitter?close` as a authorized redirect URI
2. Run `PATH_TO_CHECKOUT/meteor add accounts-twitter`
3. Add the following line to a file visible to both client and server, e.g. `accounts/providers.js`:
```
Meteor.accounts.twitter.config(CONSUMER_KEY, APP_URL);
```

4. Add the following line to a file visible only to the server, e.g. `accounts/server/provider_secrets.js`:
```
Meteor.accounts.twitter.setSecret(CONSUMER_SECRET);
```


#### Getting a Twitter consumer key and secret (step 1 above)
1. Go to https://dev.twitter.com/apps/new
2. Callback URL should be APP_URL/_oauth/twitter?close
