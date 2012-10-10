## Introduction

We now have the first chunk of our authentication implemented on the
`auth` branch in GitHub (https://github.com/meteor/meteor/tree/auth).

Here are the high-level changes:

1. As part of livedata, method and subscription functions on the
server now have access to the current user ID with `this.userId`.
This means you can limit what a method does and what data a publish
function sends to the client based on each client's login state.
Since subscriptions are long-lived, Meteor reruns a client's
subscriptions when its user ID changes (eg login or logout).

2. The "accounts-base" smart package defines a new Meteor.Collection called
`Meteor.users`.  We've written several packages that manage this collection
using both third party services such Facebook and Google (via OAuth2)
and old fashioned usernames and passwords.

3. The "accounts-ui" smart package provides convenient chrome on the
client for a login form `{{> loginButtons}}`. It also provides a UI
to help configure third party login services.

As an example of these new features, we've added "private" items to
the Todos example, which can be seen at http://auth-todos.meteor.com.
Each user who is logged into todos can now mark items as private,
which no other user can see or modify.

## Getting Started

### Warning
The `auth` branch is a work in progress. The features and API may change at any time.

### Adding Accounts to your app
1. [Get Meteor running from a git checkout](https://github.com/meteor/meteor#slow-start-for-developers). Checkout the `auth` branch.
2. Run `PATH_TO_CHECKOUT/meteor add accounts-ui`
3. Add `{{> loginButtons}}` somewhere in your app. This adds login buttons for whatever services you configure.
4. Add login services -- see below (e.g. `PATH_TO_CHECKOUT/meteor add accounts-google accounts-facebook accounts-password`)
5. Restrict writes using `Collection.allow` and `Collection.deny` (see below. here's [what we did for todos](https://github.com/meteor/meteor/blob/171816005fa2e263ba54d08d596e5b94dea47b0d/examples/todos/server/access_control.js))
6. You probably want to turn off the autopublish (if you want to control which users see which data): `PATH_TO_CHECKOUT/meteor remove autopublish` 
7. To make collections read-only by default, `PATH_TO_CHECKOUT/meteor remove insecure`

### Updates to the API
#### Basics
- [Client/Server] `Meteor.users` is a collection of all users. By default the current user's data fields (eg "emails", "profile" and "username") are published to all clients. If autopublish is enabled all public fields of all users are published. You can choose to publish any additional fields -- overlapping subscriptions should work fine.
- [Client/Server] Within methods/subscriptions -- `this.userId` returns the current user ID
- [Client/Server(methods)] `Meteor.user()` is a reactive function returning:
 - user document if the user is logged in and the user document data is fully loaded on the client
 - `null` if the user is logged out
 - `{_id: (user id)}` if the user is logged in but we are still waiting for the subscription to load on the client. You can differentiate this from the logged-in state by using the reactive function `Meteor.userLoaded()`, which returns true if the user is logged in and the user document data is fully loaded
- [Client/Server(methods)] `Meteor.userId()` is a reactive function that returns the current user id.
- [Client] A global Handlebars helper named `currentUser` (e.g. `{{#if currentUser}}Make private{{/if}}`), and one named `currentUserLoaded` (equivalent to `Meteor.userLoaded()`)

#### Configuration
[Client/Server] `Accounts.config(options)` - Global configuration of the accounts system. Affects both the low-level API and the appearance of `accounts-ui`. 

NOTE: We are fairly confident that this API will change.

Options:
- `requireEmail` (Boolean) - Require users created to have an email. This also effects the login and signup forms in `accounts-ui`
- `requireUsername` (Boolean) - Require users created to have a username. This also effects the login and signup forms in `accounts-ui`
- `sendConfirmationEmail` (Boolean) - Send email address confirmation emails to users created via `accounts-password`.

#### Disabling full write access
By default, clients are given full write access to all collections. To turn this behavior off, remove the `insecure` package

#### Restricting writes
[Server] `collection.allow(options)` and `collection.deny(options)`. Restricts default write methods on this collection. Once either of these are called on a collection, all write methods on that collection are restricted regardless of the `insecure` package.

Allow and deny can be called multiple times. The validators are
evaluated as follows:
- If any deny() function returns true, the request is denied.
- Otherwise, if any allow() function returns true, the requested is allowed.
- Otherwise, the request is denied.


Options:
- `insert` (Function(userId, doc)) - Return true to allow/deny adding this document
- `update` (Function(userId, docs, fields, modifier)) - Return true to allow/deny updating these documents.
 - `fields` - Array of fields to be modified
 - `modifier` - Original mongo [modifier operation](http://www.mongodb.org/display/DOCS/Updating#Updating-ModifierOperations)
- `remove` (Function(userId, docs)) - Return true to allow/deny removing these documents
- `fetch` (Array) - Fields to be fetched for update and remove restrictions (if not passed, all fields will be fetched)

#### Email templates

[Server] `Accounts.emailTemplates` - An object that can be modified to customize the emails that are sent.

#### Low-level API
If you're not using `accounts-ui` or `accounts-ui-unstyled`, use these functions to implement your own login flow. You'll also have to handle the special URLs sent in emails by showing dialogs for email confirmation, reset password and account enrollment. You can also use `accounts-ui` without `{{> loginButtons}}` if you just want to get the dialogs.

- [Client] `Meteor.loginWithFacebook(callback)`
- [Client] `Meteor.loginWithGoogle(callback)`
- [Client] `Meteor.loginWithWeibo(callback)`
- [Client] `Meteor.loginWithTwitter(callback)`
 - `callback`: Function(error|null).
  - If error is an instance of Accounts.ConfigError, you're missing the appropriate configuration document in mongo (see "Reconfiguring login services" below).
  - If error is an instance of Accounts.LoginCancelledError, the user closed the login pop-up or didn't agree to give permissions to your app
  - Otherwise, it could be either an expected server error (e.g., if you used Accounts.validateNewUser and the proposed user document doesn't validate), or an unexpected server error.

- [Client] `Meteor.loginWithPassword(user, password, callback)`
 - `user` argument is either `{username: 'username'}`, `{email: 'email@address'}`, or a string that might be username or email.
 - `password`: the plaintext password. The password is _not_ sent unencrypted, though.
 - `callback`: Function(error|null)
- [Client] `Meteor.logout(callback)`
 - `callback`: Function(error|null)
- [Client] `Accounts.createUser(options, extra, callback)` - Creates a user and logs in as that user
 - `options` a hash containing: `username` and/or `email`, `password`
 - `extra`: extra fields for the user object (eg `name`, etc); by default only `profile` and its subfields are allowed.
 - `callback`: Function(error|null)
- [Server] `Accounts.createUser(options, extra)` - Creates a user. If email is specified and password is not, sends that user an email with a link to choose their initial password and complete their account enrollment
 - `options` a hash containing: `email`, `username`, and/or `password`
 - `extra`: extra fields for the user object (eg `name`, etc); by default only `profile` and its subfields are allowed.
 - returns the newly created user id.
- [Client] `Accounts.changePassword(oldPassword, newPassword, callback)`
 - `callback`: Function(error|null)
 - Must be logged in to call this. Changes the currently logged in user.
- [Client] `Accounts.forgotPassword(options, callback)` - Requests that a reset password link be sent to a user
 - `options`: object containing an `email` field
 - `callback`: Function(error|null)
- [Client] `Accounts.resetPassword(token, newPassword, callback)` - Resets a user's password
 - `token`: unique string contained in the email sent to the user by `Accounts.forgotPassword`
 - `callback`: Function(error|null)
- [Server] `Accounts.setPassword(userId, newPassword)` - Force change a user's password on the server.
- [Client] `Accounts.confirmEmail(token, callback)` - Confirm a user's email
 - `token`: unique string contained in the email sent to the user by a client-side call to `Accounts.createUser` (in case `sendConfirmationEmail` was set to true in the call to `Accounts.config`)
 - `callback`: Function(error|null)


#### Configuring login services
- [Client/Server] `Accounts.facebook.config(options)`
- [Client/Server] `Accounts.google.config(options)`

Options:
- scope: a list of permissions to request when logging in. For example, on Facebook: `scope: ["user_birthday", "user_checkins"]`. On Google: `scope: ["https://www.googleapis.com/auth/calendar"]`


#### Controlling new user creation

- [Server] `Accounts.validateNewUser(validator)`
 - `validator`: Function(proposedUser). returns true to allow user creation, false to deny.
 - Can be called multiple times. All validators must pass for the user to be created.
- [Server] `Accounts.onCreateUser(Function(options, extra, user))`
 - `options`: Basic parameters for user creation. eg `username`, `email`.
 - `extra`: extra fields proposed for the new user. straight from the client. eg `name`.
 - `user`: A pre-processed user object with transformed options.
 - return value of function: a proposed user object, with all fields filled out. This can be based on the user object passed in, or constructed totally differently. throw an error to abort user creation.
 - This can only be set once. If it is not set, the default implementation simply copies the `profile` field from `extra into the user object, and throws an error if there are other fields in `extra`.

### Reconfiguring login services
`accounts-ui` supplies a simple way to configure external login services, but here's what happens underneath the hood:
- There is a new `Accounts.loginServiceConfiguration` collection (the Mongo collection name is `meteor_accounts_loginServiceConfiguration`). It contains documents like: 
```js
{ 
  "service" : "twitter", 
  "consumerKey" : "8RATKYCENSORED",
  "secret" : "9hLdJD5OFXkkmKaathzXfdYSBgZ3NUgPwHCENSORED",
  "_id" : "5b7aceca-404a-4288-882f-f910b117bd2f"
}
```
- The easiest way to reconfigure a login service is to remove that document using the mongo shell (`meteor mongo [<domain name>]`, `db.meteor_accounts_loginServiceConfiguration.remove({service: "twitter"})`) and reconfigure using accounts-ui. Alternatively you could update the Mongo document directly.