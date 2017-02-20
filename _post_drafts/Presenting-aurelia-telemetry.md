---
layout: post
title: "Presenting aurelia-telemetry"
tags: [Aurelia, JS, TypeScript, Web]
image:
  feature: "aurelia-main-logo.svg"
  teaser: "aurelia-main-logo.svg"
published : true
--- 

Adding telemetry to an Aurelia application is a tedious and repeatable task, at least when it comes to
bootstrapping and integrating it with the framework. This is the problem the 
[`aurelia-telemetry`](https://github.com/manuel-guilbault/aurelia-telemetry) plugin intends to solve.

## Features

The plugin defines an abstraction for a telemetry client, and integrates it with Aurelia so all different 
kinds of telemetry are automatically gathered:

* Logical page views
* Unhandled errors
* Logs

Additionally, the plugin defines a `trackEvent` binding behavior, which can be added to any `.delegate`,
`.trigger`, or `.call` binding, so a custom event is tracked when the binding is triggered:

```html
<button click.delegate="doSomething() & trackEvent:'something-happened'">Do something</button>
```

Here, each time the button is clicked, a `'something-happened'` event will be tracked.

The binding behavior can also be passed an object containing additonal properties as its second parameter,
and this object will be sent to the telemetry client along with the event name:

```html
<button click.delegate="doSomething() & trackEvent:'something-happened':{ param: 'value' }">Do something</button>
```

## Adapters

The plugin doesn't contain a specific telemetry client. Instead, it just defines a `TelemetryClient` abstract class.
This means you can plug an adapter for your favorite telemetry framework. Alternatively, you can use one of the
existing adapters already provided:

* [`aurelia-telemetry-application-insights`](https://github.com/manuel-guilbault/aurelia-telemetry-application-insights)
* [`aurelia-telemetry-google-analytics`](https://github.com/manuel-guilbault/aurelia-telemetry-google-analytics)
* aurelia-telemetry-piwik (coming soon)

## Contributing

Help is welcome, so if you want to create an adapter for another telemetry tool, or propose a new feature, don't hesitate
to either send a pull request or [open discussion](https://github.com/manuel-guilbault/aurelia-telemetry/issues/new).