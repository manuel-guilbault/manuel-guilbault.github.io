---
layout: post
title: "Presenting aurelia-telemetry"
tags: [Aurelia, JS, Web]
image:
  feature: "aurelia-main-logo.svg"
  teaser: "aurelia-main-logo.svg"
published : true
--- 

Adding telemetry to an Aurelia application is a tedious and repeatable task, at least when it comes to
bootstrapping and integrating with the framework. This is what the 
[`aurelia-telemetry`](https://github.com/manuel-guilbault/aurelia-telemetry) plugin intends to solve.

## Features

The plugin defines an abstraction for a telemetry client, and integrates it with Aurelia so all different 
kinds of telemetry are automatically gathered:

* **Logical page views**: by integrating with `aurelia-router`, the plugin is able to track when a user 
  navigates to a route.
* **Unhandled errors**: by listening for the `error` event on the `window` object, the plugin is able
  to track unhandled errors.
* **Logs**: by integrating with `aurelia-logging`, the plugin is able to track log messages.

Additionally, the plugin defines a `trackEvent` binding behavior, which can be added to any `.delegate`,
`.trigger`, or `.call` binding, so a custom event can be tracked when the binding is triggered:

```html
<button click.delegate="doSomething() & trackEvent:'something-happened'">Do something</button>
```

Here, a `'something-happened'` custom event is tracked each time the button is clicked.

The binding behavior can also be passed an object containing additonal properties as its second parameter,
and this object will be sent to the telemetry client along with the event name:

```html
<button click.delegate="doSomething() & trackEvent:'something-happened':{ param: 'value' }">Do something</button>
```

## Adapters

The plugin doesn't contain any specific telemetry client. Instead, it just defines a `TelemetryClient` abstract class,
which you can extend into an adapter for your favorite telemetry framework. Alternatively, you can use one of the
existing adapters, provided as additional plugins:

* [`aurelia-telemetry-application-insights`](https://github.com/manuel-guilbault/aurelia-telemetry-application-insights)
* [`aurelia-telemetry-google-analytics`](https://github.com/manuel-guilbault/aurelia-telemetry-google-analytics)
* `aurelia-telemetry-piwik` (coming soon)
* `aurelia-telemetry-logstash` (coming soon)

## Contributing

Of course, help is welcome. If you want to create an adapter for another telemetry provider, or propose a new feature, don't hesitate
to either send a pull request or [engage discussion](https://github.com/manuel-guilbault/aurelia-telemetry/issues/new).
