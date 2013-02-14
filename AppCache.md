More detailed information about the browser appplication cache and the
appcache package.


## About the App Cache

The application cache is an HTML5 feature which allows the static
resources for a web application (HTML, Javascript, CSS, images)
to be saved in the browser.

* The web page loads faster because the browser doesn't need to contact
  the server first.

* Hot code pushes are loaded by the browser in the background while the
  app continues to run.  Once the new code has been fully loaded the
  browser is able to switch over to the new code quickly.

* The application cache allows the application to be loaded even when
  the browser doesn't have an Internet connection, and so enables using
  the app offline.

(Note however that the app cache by itself doesn't do anything to make
*data* available offline: in an application loaded offline, a Meteor
Collection will appear to be empty in the client until the Internet
becomes available and the browser is able to establish a livedata
connection).


## Warning

Do not use the appcache package for users of an app published on a
domain if there is a chance that you might want to revent to a version
of Meteor older than 0.5.5 on that domain.

Meteor 0.5.4 and below does not return a 404 for the app manifest
file, so your users who have the app cached will be stuck running your
old code out of their app cache *even though you aren't using the
appcache package any more*.

Meteor releases 0.5.5 and later don't have this problem.


## Browser Support

The app cache is widely supported by browsers
(http://caniuse.com/#feat=offline-apps).  Older browsers will ignore
the manifest attribute and the app will run as a regular online
application.

When an app cache is enabled Firefox will pop up a message "This
website is asking to store data on your computer for offline use" and
will ask the user whether to allow or deny the request.

While many users will understand what this means, some users may
possibly be alarmed by the browser warning, and could choose not to
run the app in preference to enabling something they don't understand.

The goal of the appcache package is to work transparently: you should
be able to add the appcache package without it changing the behavior
of your application in supported browser.  Since this is not possible
on Firefox, the app cache is disabled in Firefox by default.

There are two mechanisms for configuring which browsers you'd like to
enable app cache support for.  You can specify particular browsers to
enable or disable:

````
Meteor.AppCache.config({
  firefox: true,
  IE: false
});
````

this says to enable the app cache for Firefox, to disable it for IE,
and to use the defaults for the other browsers.

You can also give an explicit list of browsers to enable:

````
Meteor.AppCache.config({
  browsers: ['chrome', 'android']
});
````

this enables the app cache for Chrome and Android, and disables it for
all other browsers.

The available browsers and their default enabled/disabled
configuration:

* `android` (enabled)
* `chrome` (enabled)
* `firefox` (disabled)
* `IE` (enabled)
* `mobileSafari` (enabled)
* `opera` (enabled)
* `safari` (enabled)

Note that even if a browser is enabled the app cache may still not be
used: the user may be using an older version of a browser which
doesn't support an app cache, or in Firefox the user can decline the
request to enable offline support.


## How the Browser Uses the App Cache

A key to understanding how the browser uses the application cache is
this:

*The browser always loads the app cache in the background.*

Or, to put it another another way, the browser never waits for the app
cache to be updated.

For example, consider what happens when a user navigates to the app's
web page for the first time, when they don't have the application
cached yet.  The browser will load the app as if it were a standard
online application not using an app cache; and then the browser will
also populate the app cache in the background.  The *second* time the
user opens the web page the browser load the app from the cache.  Why?
Because the browser never waits for the app cache to loaded.  The
first time the user opens the page the cache hasn't been loaded yet,
and so the browser loads the page incrementally from the server as it
does for web pages that aren't cache enabled.

As another example, suppose the user has previously opened the web
page and so has an old copy of the application in the app cache.  The
user now returns to the web page, and in the meantime a new version of
the application has been published.  What happens?  Since the browser
never waits for the app cache, it will at first display the old
version of the web page.  Then, as Meteor makes its livedata
connection and sees that a code update is available, the page will
reload with the new code.

This behavior may seem strange.  Why not check first to see if new
code is available and avoid potentially briefly displaying an old
version of the app?  But consider if the user is offline, or has a bad
or intermittent Internet connection.  We don't know *how long* it will
take to discover that a new version of the app is available.  It could
be five seconds or a minute or an hour... depending on when the
browser is able to connect.  So rather than waiting, not knowing how
long the wait may be, instead the browser enables using the
application offline by loading the application from the cache, and
then updates the cache when a new version is available and can be
downloaded.


## The App Cache and Meteor Code Reloads

The appcache package is designed to support Meteor's hot code reload
feature.  (If you see any problems with code reloads when using the
app cache, please report it as a bug).

With the appcache package installed a code reload will follow these
steps:

* Meteor's livedata stream connection notices that a code update is
  available and initiates a reload.

* The appcache package is notified that a code migration has started
  (as the appcache package plugs into Meteor's reload `onMigrate`
  hook).  The appcache package calls
  `window.applicationCache.update()` to ask the browser to update the
  app cache.  The appcache package then reports back to reload that it
  isn't ready for migration yet... until the browser reports that the
  app cache has finished updating.  The reload is thus delayed until
  the new code is in the cache.

* Meteor's reload calls `window.location.reload()`, which reloads
  the app in the web page with the new code in the app cache.


## Designed for Static Resources Only

When a browser displays an image in an application, we can describe
that image as either being static (it stays the same for any
particular version of the application) or dynamic (it can change as
the application is being used).

For example, a "todo" app might display a green checkmark image for
completed todo items.  This would be a static image, because it is
part of the application and changing the image would require a code
push.

Conversely, we might imagine that the app could allow the user to
upload images to add to a todo item's description.  These images would
be dynamic images, because new images can be added and images can be
images as the application is running.

The appcache package is only designed to cache static resources.  As
an "application" cache, it caches the resources needed by the
application, including the HTML, CSS, Javascript and files published
in the public/ directory.

Different browsers have different limits on the size of the
application cache, and generally respond poorly to going over the
limit.  To the application, going over the limit results in a cache
update error which is indistinguishable from the user merely not
having an Internet connection at the moment.  The cache update failure
then causes the browser to use the *old* outdated cache, which means
the application not only will not work offline but is broken online as
well.

Thus when using the appcache package we recommend keeping the static
size of the client application resources including the public/
directory under 5MB.  The appcache package will print a warning if the
total size of the resources being added to the app cache goes over
this.
