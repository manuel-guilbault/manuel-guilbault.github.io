---
layout: post
title: "Multiple Aurelia apps in a single ASP.NET Core application"
tags: [Aurelia, JS, Web, .NET]
image:
  feature: "aurelia-main-logo.svg"
  teaser: "aurelia-main-logo.svg"
published : true
---

I've been recently working on an ASP.NET Core MVC application for a tour operator company. The application follows the 
standard MVC navigation principles:

1. You search for flights between two cities, for a specific time period;
2. You see a grid with the flights matching the criteria, displaying all sorts of yielding information (sold seats, capacity, pricing parameters, etc.);
3. You click on a single flight to go to its edition page;
4. You edit the flight's capacity and pricing parameters, save, and go back to the result grid.

The customers asked if we could redesign this workflow so they could edit the pricing parameters directly in the grid. To support this demand, the grid 
being a simple `table` element rendered using a Razor template, we decided to transform it into a small Aurelia application which would render the grid 
and handle inline edition. 

As the customer also mentionned they would like to have some other workflows redesigned as such in the future, we figured
we would end up with multiple Aurelia apps in our .NET project. As such, we wanted modified the default setup of an ASP .NET Core Aurelia app
to support multiple client apps.

We needed to fiddle a bit here and there, but we ended up being able to do it pretty easily. Here's a recap of how we did it.

## Setting up the project & the environment

To follow this example, you'll need to create a .NET Core Aurelia web app:

1. Install the [.NET Core CLI](https://www.microsoft.com/net/core)
2. Install the ASP.NET Spa Templates: `dotnet new --install "Microsoft.AspNetCore.SpaTemplates::*"`
3. Create a new folder for your project: `mkdir dotnet-core-multiple-aurelia-apps`
4. Move into it: `cd dotnet-core-multiple-aurelia-apps`
5. Create your ASP.NET Core Aurelia project: `dotnet new aurelia`

> Of course, since Aurelia is involved, you'll also need to have [NodeJS](https://nodejs.org/) installed.

## Transforming a single Aurelia app project into a multiple one

Once you got your ASP.NET Core project ready, you can start transforming it into a multiple Aurelia apps project.
Quickly:

1. rename the `ClientApp` directory to `ClientApps`;
2. create `ClientApps/app1`, where `app1` is the name of your first Aurelia app;
3. move everything from `ClientApps` inside `ClientApps/app1`.

Next, in `webpack.config.js`:

1. replace every occurences of `ClientApp` with `ClientApps`;
2. change the file so it returns one configuration object per Aurelia app instead of a single one;
3. for each app's configuration object:
  - change the `entry` point's name from `'app'` to the app's name;
  - change how it `resolve`s its modules, not from `ClientApps` anymore, but from the app directory inside `ClientApps`.

The updated `webpack.config.js` should look like this:

```js
const path = require('path');
const webpack = require('webpack');
const { AureliaPlugin } = require('aurelia-webpack-plugin');
const bundleOutputDir = './wwwroot/dist';

module.exports = (env) => {
    const isDevBuild = !(env && env.prod);
    return [
        'app1',
        // Add any other Aurelia app here
    ].map(appName => ({
        stats: { modules: false },
        entry: { [appName]: 'aurelia-bootstrapper' },
        resolve: {
            extensions: ['.ts', '.js'],
            modules: [`ClientApps/${appName}`, 'node_modules'],
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
    }));
}
```

> This configuration will now generate one bundle for each application. For exemple, the `app1` application will be bundled 
  in the `wwwroot/dist/app1.js` file.

Lastly, load the proper bundles and bootstrap your Aurelia app in the proper Razor view:

```html
<div aurelia-app="boot">Loading...</div>

@section scripts {
  <script type="text/javascript" src="~/dist/vendor.js" asp-append-version="true"></script>
  <script type="text/javascript" src="~/dist/app1.js" asp-append-version="true"></script>
}
```

> Before you run the application, make sure to generate the `vendor` bundle by running `webpack --config webpack.config.vendor.js`.

Check this [GitHub repository](https://github.com/manuel-guilbault/blog-post-aspnet-core-multiple-aurelia-apps) for a code sample.
