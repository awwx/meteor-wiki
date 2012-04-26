### Help

Visit [Stackoverflow.com](http://stackoverflow.com/questions/tagged/meteor), or the IRC channel(freenode.com).

### Contributing code to the Meteor project

The best way (the only way?) to contribute code is to submit a pull request.  We have a few guidelines for you.

* Development happens on the **devel** branch, so please make sure your branch is based off that.
* Sign the [contributor's agreement](http://contribute.meteor.com/).
* Follow the [Meteor style guide](https://github.com/meteor/meteor/wiki/Meteor-Style-Guide).


### Submitting a new package

#### Pre-processors

We love new pre-processor packages. Here are some guidelines for submitting a new pre-processor:

* It should have tests. See `packages/coffeescript` or `packages/less` for examples.

* It should be documented. See `docs/client/packages`.

* It should be a stable version. If it requires an npm module install, make sure to add it to `admin/generate-dev-bundle.sh` with a fixed version number. If the preprocessor is only a few files, you can avoid the dev bundle and simply put the code in the package directory.


#### Simple Assets

Because the package API is still in flux, and because there are so many client-side libraries out there, we're not taking new packages that only include files to be shipped to the client at this time. You can easily get the same effect by placing the files in your application source tree. Place client library files in the directory `client/lib/`.

### Other changes

If you're working on a big ticket item, best to check in first on the #meteor IRC channel or email us at contact@meteor.com.  We'd hate to have to steer you in a different direction after you've already put in a lot of hard work.
