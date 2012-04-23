Solved any common Meteor question? Put the answer in here!

## What do I need to use meteor? Do I need node.js or mongodb?

No! Just follow [the quick start guide](https://github.com/meteor/meteor). Node and Mongo come bundled with Meteor.

## I heard the database isn't secured, is this true?

Yes and no...future versions of meteor will have more support in general for authorization. And, out of the box it is true that the client has full access to the server.

But, there are two ways you can secure your meteor installation:

1. Limit [the database methods](http://stackoverflow.com/questions/10115042/how-do-you-secure-the-client-side-mongodb-api) that the client has access to.
2. Wrap your sensitive calls in Meteor.method/Meteor.call so that the client must call the server to obtain access.

## How can I use Meteor.methods to handle async code on the server side?

First: if you're doing a Meteor.call on the client side you can pick your poison and make calls to your server either synchronously or asynchronously.

The server method definition, however, supports synchronous handling. Look [at stackoverflow](http://stackoverflow.com/questions/10251130/how-to-wait-for-sub-process-results-before-returning-from-meteor-method) for ways around this, if you need them.

## Where do I have to place my image files?
You have to put them into the /public folder (you may have to create it). 

## What's the best way to add coffeescript, less, ... to my project?
See http://docs.meteor.com/#packages
