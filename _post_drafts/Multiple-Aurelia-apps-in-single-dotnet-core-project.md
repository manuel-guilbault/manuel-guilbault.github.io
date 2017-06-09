---
layout: post
title: "Multiple Aurelia apps in a single ASP .NET Core application"
tags: [Aurelia, JS, Web, dotNet]
image:
  feature: "aurelia-main-logo.svg"
  teaser: "aurelia-main-logo.svg"
published : true
---

I've been recently working on an ASP .NET Core MVC application for a tour operator company. The application follows the 
standard MVC navigation principles:
1. You search for flights between two cities, for a specific time period;
2. You see a grid with the flights matching the criteria, displaying all sorts of yielding information (sold seats, capacity, pricing parameters, etc.);
3. You click on a single flight to go to its edition page;
4. You edit the flight's pricing parameters, save, and go back to the result grid.

The customers asked if we could redesign this workflow so they could edit the pricing parameters directly in the grid. To support
this demand, we wanted to use a small Aurelia application to handle the grid.

As we may need to add other small apps in the future, we tried to modify the default setup of an ASP .NET Core Aurelia app
to support multiple Aurelia apps. Here's how we did it:

1. Rename `ClientApp` to `ClientApps`;
2. Create `ClientApps/app1`, where `app1` is the name of your small Aurelia app;
3. Move everything from `ClientApps` inside `ClientApps/app1`;
4. Change `webpack.config.js` so it returns one bundle configuration per Aurelia app instead of a single one:

```js
const path = require('path');
const webpack = require('webpack');
const { AureliaPlugin } = require('aurelia-webpack-plugin');
const bundleOutputDir = './wwwroot/dist';

module.exports = (env) => {
    const isDevBuild = !(env && env.prod);

    // Add other Aurelia applications in this array:
    return [
        'app1'
    ].map(name => {
        var entry = {};
        entry[name] = 'aurelia-bootstrapper';

        return {
            stats: { modules: false },
            entry,
            resolve: {
                extensions: ['.ts', '.js'],
                modules: [`ClientApps/${name}`, 'node_modules']
            },
            output: {
                path: path.resolve(bundleOutputDir),
                publicPath: '/dist/',
                filename: '[name].js'
            },
            module: {
                rules: [
                    { test: /\.ts$/i, include: /ClientApps/, use: 'ts-loader?silent=true' },
                    { test: /\.html$/i, use: 'html-loader' },
                    { test: /\.css$/i, use: isDevBuild ? 'css-loader' : 'css-loader?minimize' },
                    { test: /\.(png|jpg|jpeg|gif|svg)$/, use: 'url-loader?limit=25000' }
                ]
            },
            plugins: [
                new webpack.DefinePlugin({ IS_DEV_BUILD: JSON.stringify(isDevBuild) }),
                new webpack.DllReferencePlugin({
                    context: __dirname,
                    manifest: require('./wwwroot/dist/vendor-manifest.json')
                }),
                new AureliaPlugin({ aureliaApp: 'boot' })
            ].concat(isDevBuild ? [
                new webpack.SourceMapDevToolPlugin({
                    filename: '[file].map', // Remove this line if you prefer inline source maps
                    moduleFilenameTemplate: path.relative(bundleOutputDir, '[resourcePath]')  // Point sourcemap entries to the original file locations on disk
                })
            ] : [
                new webpack.optimize.UglifyJsPlugin()
            ])
        };
    });
};
```

This configuration will generate one bundle for each application. For exemple, our `app1` application will be bundled in the
`app1.js` file. If the application also has CSS files, it will be bundled in `app1.css`.

5. Load vendors + app bundles & bootstrap Aurelia app in the proper Razor view:
```html
  <div aurelia-app="boot">Loading...</div>
  <script type="text/javascript" src="~/dist/vendor.js" asp-append-version="true"></script>
  <script type="text/javascript" src="~/dist/app1.js" asp-append-version="true"></script>
```
Of course, if `app1` has CSS content, you'll also need to load the CSS bundle.

Check this [GitHub repository]() for a code example.
