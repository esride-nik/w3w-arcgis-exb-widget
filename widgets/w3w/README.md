# what3words widget
## by [Niklas Köhn](https://github.com/esride-nik), Esri Deutschland

This widget uses the what3words REST API to find the w3w address of the click point resp. the center point of the map.

## How to use the sample
Clone the sample repo into your Experience Builder Client root folder and restart your watcher. On the configuration pane, you'll need to enter your what3words API key. Don't worry, it's free! [So go ahead!](https://developer.what3words.com/public-api)

## How it works
Connecting widget to configured map widget in the event handler `onActiveViewChange` follows the usual Experience Builder developement pattern for connecting datasources. The what3words widget also creates the necessary layer objects, event listeners and property watchers. Depending on the configuration, the widget reacts to map clicks (`handleMapClick`) or takes action when the map stops moving (`stationaryWatchHandler`). 

While map graphics are drawn manually by the widget code, the widget itself is a React component that redraws autoamtically after the `setState` method has run and updated the observable data in the widget state.


## Polyfills for Node APIs in Experience Builder >= 1.8

Webpack 5 no longer includes polyfills for node core modules. This manifests in the error message ``BREAKING CHANGE: webpack < 5 used to include polyfills for node.js core modules by default``, meaning: Packages that use node.js core modules no longer work natively with webpack 5.

For more information on this topic, here is a good explanation: "[Webpack 4 automatically polyfilled many node APIs in the browser. This was not a great system, because it could lead to surprisingly giant libraries getting pulled into your app by accident, and it gave you no control over the exact versions of the polyfills you were using. So Webpack 5 removed this functionality.](https://gist.github.com/ef4/d2cf5672a93cf241fd47c020b9b3066a)"

There are many descriptions for the workaround, directly in the error messages in the console and also under the above mentioned source: The required sources have to be entered in ``webpack.config`` under ``resolve.fallback`` and installed via ``npm i``.

So far so good. Except that this configuration has been chopped up in ArcGIS Experience Builder and spread over several files:

![Webpack config files in ExB](./assets/webpack_configs.png)

### From left to right:
* ``webpack.config.js`` in the ``client`` root folder referencing ``webpack-extensions.config.js`` in the ``webpack`` subfolder.
* ``webpack-extensions.config.js`` builds the config from references to ``webpack-extensions.common.js``
* ``webpack-extensions.common.js`` references the actual polyfills in ``webpack.common.js``.

### Steps to do:
* In ``webpack-extensions.common.js``, add a ``fallback`` property to the ``resolve`` object in 3 places: ``getTemplatesWebpackConfig()``, ``getWidgetsWebpackConfig()`` and ``getThemesWebpackConfig()``:
    ```
    resolve: {
        alias: webpackCommon.moduleAlias,
        extensions: webpackCommon.extensions,
        mainFields: webpackCommon.resolveMainFields,
        fallback: webpackCommon.fallback
    },
    ```
* In ``webpack.common.js``, add the references to the polyfilled packages under ``exports.fallback``, e.g.:
    ```
    exports.fallback = {
        "os": require.resolve("os-browserify/browser"),
        "util": require.resolve("util/"),
        "http": require.resolve("stream-http"),
        "url": require.resolve("url/"),
        "stream": require.resolve("stream-browserify"),
        "https": require.resolve("https-browserify"),
        "zlib": require.resolve("browserify-zlib"),
        "assert": require.resolve("assert/"),
        "buffer": require.resolve("buffer/"),
    }
    ```
* ToDo: Install all referenced NPM packages on ``client`` level, e.g.:
    ```
    npm i os-browserify
    npm i util
    npm i stream-http
    npm i stream-browserify
    npm i https-browserify
    npm i browserify-zlib
    npm i buffer
    npm i assert
    npm i url
    ```

That works.

The trouble is, that the webpack files on root level of the "client" folder are not part of the "exb-web-extension-repo", but are delivered with ExB. Now if you use NPM packages in your custom widgets that require polyfills, you have to update the webpack configs on Node JS root level. Please refer to the ``_webpack5-config-updates`` subfolder in this repository for sample files.

[Originally published in the ArcGIS community.](https://community.esri.com/t5/arcgis-experience-builder-questions/npm-packages-in-experience-builder-1-8/m-p/1181885#M4574)