---
title: Customizing web app initialization
description: Customize how Flutter apps are initialized on the web
---

You can customize how a Flutter app is initialized on the web
using the `_flutter.loader` JavaScript API.
This API can be used to display a loading indicator in CSS,
prevent the app from loading based on a condition,
or wait until the user presses a button before showing the app.

The initialization process is split into the following stages:

**Loading the entrypoint script**
: Fetches the `main.dart.js` script and initializes the service worker.

**Initializing the Flutter engine**
: Initializes Flutter's web engine by downloading required resources
  such as assets, fonts, and CanvasKit.

**Running the app**
: Prepares the DOM for your Flutter app and runs it.

This page shows how to customize the behavior
at each stage of the initialization process.

## Getting started

By default, the `index.html` file
generated by the `flutter create` command
contains a script tag
that calls `loadEntrypoint` from the `flutter.js` file:

```html
<html>
   <head>
      <!-- ... -->
      <script src="flutter.js" defer></script>
   </head>
   <body>
      <script>
         window.addEventListener('load', function (ev) {
             // Download main.dart.js
             _flutter.loader.loadEntrypoint({
                 serviceWorker: {
                     serviceWorkerVersion: serviceWorkerVersion,
                 }
             }).then(function (engineInitializer) {
                 // Initialize the Flutter engine
                 return engineInitializer.initializeEngine();
             }).then(function (appRunner) {
                 // Run the app
                 return appRunner.runApp();
             });
         });
      </script>
   </body>
</html>
```



{{site.alert.note}}
  In Flutter 2.10 or earlier,
  this script doesn't support customization.
  To upgrade your `index.html` file to the latest version,
  see [Upgrading an older project](#upgrading-an-older-project).
{{site.alert.end}}



The `loadEntrypoint` function returns a JavaScript [`Promise`][js-promise]
that resolves when the Service Worker is initialized
and the `main.dart.js` entrypoint has been downloaded by the browser.
It resolves with an **engine initializer** object
that initializes the Flutter Web engine.

The `initializeEngine()` function returns a promise
that resolves with an **app runner** object
that has a single method `runApp()` that runs the Flutter app.

[js-promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise

## Customizing web app initialization

In this section,
learn how to customize each stage of your app’s initialization.

### Loading the entrypoint

The `loadEntrypoint` method accepts these parameters:

`entrypointUrl`
: The URL of your Flutter app's entrypoint. Defaults to `main.dart.js`.

`serviceWorker`
: The configuration of the `flutter_service_worker.js` file.
  If this isn’t set, the service worker won’t be used.
      
`serviceWorkerVersion`
: Pass *the `serviceWorkerVersion` var* set by
  the build process in your <strong><code>index.html</code></strong> file.
  
`timeoutMillis`
: The timeout value for the service worker load.
  Defaults to <strong><code>4000ms</code></strong>.


### Initializing the engine

Instead of calling `initializeEngine()` on the engine initializer,
you can call `autoStart()` to immediately start the app
with the default configuration
instead of using the app runner to call `runApp()`:


```js
_flutter.loader.loadEntrypoint({
  serviceWorker: {
    serviceWorkerVersion: serviceWorkerVersion,
  }
}).then(function(engineInitializer) {
  return engineInitializer.autoStart();
});
```

## Example: Display a progress indicator

To give the user of your application feedback
during the initialization process,
use the hooks provided for each stage to update the DOM:


```html
<html>
   <head>
      <!-- ... -->
      <script src="flutter.js" defer></script>
   </head>
   <body>
      <div id="loading"></div>
      <script>
         window.addEventListener('load', function(ev) {
           var loading = document.querySelector('#loading');
           loading.textContent = "Loading entrypoint...";
           _flutter.loader.loadEntrypoint({
             serviceWorker: {
               serviceWorkerVersion: serviceWorkerVersion,
             }
           }).then(function(engineInitializer) {
             loading.textContent = "Initializing engine...";
             return engineInitializer.initializeEngine();
           }).then(function(appRunner) {
             loading.textContent = "Running app...";
             return appRunner.runApp();
           });
         });
      </script>
   </body>
</html>
```


For a more practical example using CSS animations,
see the [initialization code][gallery-init] for the Flutter Gallery.

[gallery-init]: {{site.github}}/flutter/gallery/blob/master/web/index.html

## Upgrading an older project

If your project was created in Flutter 2.10 or earlier,
you can create a new `index.html` file
with the latest initialization template by running `flutter create`.

From your project directory, run the following:

```
$ flutter create .
```

You can now customize the startup process as described above.
