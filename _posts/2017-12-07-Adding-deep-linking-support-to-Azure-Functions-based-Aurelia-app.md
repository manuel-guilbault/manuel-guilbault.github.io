---
layout: post
title: "Adding deep linking support to an Azure Functions-based Aurelia app"
tags: [Azure, VSTS, DevOps]
image:
  feature: "microsoft-azure-logo.svg"
  teaser: "microsoft-azure-logo.svg"
summary: "In the previous posts of this serie, we saw a very unexpensive solution to host an Aurelia app on Azure.
We also saw how to automate the deployment process of an Aurelia app on Azure using Visual Studio Team Services.

However, the current state of the solution doesn't support deep linking if the app uses the router in push state mode.
In this post, we'll see how to fix this."
published : true
---

In this serie:

1. [Hosting an Aurelia app on Azure](/blog/2017/08/22/Hosting-an-Aurelia-app-on-Azure/)
2. [Deploying an Aurelia app on Azure using VSTS](/blog/2017/12/04/Deploying-an-Aurelia-app-on-Azure-using-VSTS/)
3. [Adding deep linking support to an Azure Functions-based Aurelia app (this post)](/blog/2017/12/07/Adding-deep-linking-support-to-Azure-Functions-based-Aurelia-app/)
4. Adding Let's Encrypt to an Azure Functions-based Aurelia app (coming soon)

<hr>

In the previous posts of this serie, we saw a very unexpensive solution to host an Aurelia app on Azure.
We also saw how to automate the deployment process of an Aurelia app on Azure using Visual Studio Team Services.

However, the current state of the solution doesn't support deep linking if the app uses the router in push state mode.
In this post, we'll see how to fix this.

You can get the sample Aurelia app I used for this post 
[here](https://github.com/manuel-guilbault/blog-post-aurelia-azure/releases/tag/2017-12-08-Adding-deep-linking-support-to-Azure-Functions-based-Aurelia-app){:target="_blank"}.

## Understanding how deep linking works in push state mode

Imagine an Aurelia app that uses the router in push state mode. Imagine this app is deployed on 
`http://www.my-awesome-app.com/`. Imagine that a user accesses the app and, through the navigation menu, 
navigates to `http://www.my-awesome-app.com/some-feature` and bookmarks this URL.
What normally happens when he comes back to this bookmarked URL later?

1. The web server receives a GET request for `/some-feature`.
2. It searches its file system (or its storage mechanism, whatever it is) for a file matching this path,
   but can't find any matching file.
4. It returns a `200 OK` response with the content of `/index.html` instead of a `404 Not Found`.
5. Since the user's browser received `/index.html`, the Aurelia app starts.
6. Since the path of the current URL is `/some-feature` in the user's browser, the Aurelia router 
   loads the component linked to this path. Everything works as expected.

At the moment, our solution doesn't perform step 4 properly. If the proxy app receives a GET request
for a path not matching a file in the Storage container, it just returns a `404 Not Found` response.
How can we fix that?

## Replacing the proxy functions

At the moment of writing, the Azure Functions proxies don't support this type of fallback mechanism.
The only solution is to remove the `proxies.json` file and to create a full-fledged Azure Function
that will act as a proxy and that will implement the fallback mechanism.

First, delete the `azure/functions-app/proxies.json` file.

Then, in your Aurelia app root directory, adjust the file structure to match the following:

```
azure
└── functions-app
    ├── proxy
    │   ├── function.json
    │   └── index.js
    └── host.json
```

Next, put the following snippet in `azure/functions-app/proxy/function.json`:

```json
{
  "bindings": [
    {
      "name": "request",
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "anonymous",
      "methods": [ "get" ],
      "route": "{*path}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    }
  ]
}
```

This file states that the function will be triggered when the Functions app receives a GET request to
any path. The function will receive the HTTP request as its `request` parameter, and will have the
request path available as the `path` context variable.

In `azure/functions-app/proxy/index.js`, put the following code:

```js
const http = require('http');
const path = require('path');
const { parse: parseUrl, format: formatUrl } = require('url');

module.exports = function(context, request) {
  const pathname = context.bindingData.path || '';
  const backendUrl = getBackendUrl(pathname);

  sendGetRequest(backendUrl, (error, response) => {
    if (error) {
      context.log.error(`Request to ${backendUrl} failed: ${error}`);
      context.done(null, { status: 500, body: '500 Internal Server Error' });

    } else if (response.status === 404) {
      const indexUrl = getBackendUrl('index.html');
  
      sendGetRequest(indexUrl, (error, response) => {
        if (error) {
          context.log.error(`Request to ${indexUrl} failed: ${error}`);
          context.done(null, { status: 500, body: '500 Internal Server Error' });
        } else {
          context.log.info(`Request to ${indexUrl} returned ${response.statusCode}`);
          context.done(null, response);
        }
      });

    } else {
      context.done(null, response);
    }
  });
};

function getBackendUrl(pathname) {
  const storageHostAndPath = process.env['Storage.HostAndContainer'];
  const sasToken = process.env['Storage.SasToken'];

  if (pathname === '') {
    pathname = 'index.html';
  }

  const hostAndPath = path.posix.join(storageHostAndPath, pathname);
  const rawUrl = `http://${hostAndPath}${sasToken}`;
  return parseUrl(rawUrl);
}

function sendGetRequest(uri, callback) {
  const options = {
    protocol: uri.protocol,
    hostname: uri.hostname,
    port: uri.port,
    path: uri.path,
  };
  const request = http.get(options, (response) => { toAzureFunctionsResponse(response, callback); });
  request.on('error', (e) => { callback(e, null); });
  request.end();
}

function toAzureFunctionsResponse(response, callback) {
  const azureFunctionsResponse = {
    status: response.statusCode,
    headers: response.headers,
    body: ''
  };
  response.on('error', (error) => { callback(error, null); });
  response.on('data', (chunk) => {
    azureFunctionsResponse.body += chunk;
  });
  response.on('end', () => {
    callback(null, azureFunctionsResponse);
  });
}
```

This file exports a function which will be executed by the Azure Functions runtime based on the triggers
it is bound to in the `function.json` file. It first retrieves the request `path` from its execution context,
and computes the URL to the Storage container for this path using the `Storage.HostAndContainer` and
`Storage.SasToken` app settings. It then forwards the request to this URL. If the path is found on the
Storage container, the response is piped back to the client. If the Storage container returns a `404 Not Found`,
the function then falls back to `/index.html` instead.

Lastly, we need to change the `azure/functions-app/host.json` file. By default, Azure Functions HTTP triggers
will match only routes with the `/api` prefix. We just need to remove this default prefix:

```json
{
  "http": {
    "routePrefix": ""
  }
}
```

You can now redeploy this app (this should be easy if you followed my 
[previous post](/blog/2017/12/04/Deploying-an-Aurelia-app-on-Azure-using-VSTS/)).
If you give it a try, deep linking should now work properly.

## Conclusion

Solving the deep linking problem was not that complicated. However, it would be pretty neat if 
the Azure Functions proxies supported this kind of feature (there's already a 
[feature request](https://github.com/Azure/Azure-Functions/issues/606){:target="_blank"} for this).
In the meantime, this work around is an okay enough solution.

## What's next?

In my next post, we'll see how to add a custom domain to our app. We'll also see how to integrate the
[Let's Encrypt Azure site extension](https://github.com/sjkp/letsencrypt-siteextension){:target="_blank"} with our
Azure Functions app, so we can enable HTTPS by generating an SSL certificate for free using
[Let's Encrypt](https://letsencrypt.org/){:target="_blank"}.
