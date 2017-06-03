---
layout: post
title: "Aurelia - feedback from the trenches"
tags: [Aurelia, JS, Web]
image:
  feature: "aurelia-main-logo.svg"
  teaser: "aurelia-main-logo.svg"
published : true
---

I've been using [Aurelia](http://aurelia.io/){:target="_blank"} to write multiple applications of 
various size for more than a year now, and I thought it might interest some people to have some feedback.

All in all, the framework is awesome. The developer experience is much better with Aurelia than with 
other frameworks I've used (\**cough*\* Angular \**cough*\*): the framework is flexible, extensible, 
and very little intrusive. You don't have to fight with it to do things the right way, whatever that
right way is.

However, entropy is constantly at work, and things can quickly become messy if you don't pay attention.
Here are a few things I've learnt along the way.

## Type everything

If you don't use [TypeScript](https://www.typescriptlang.org/){:target="_blank"} yet, you really should.
Its type system adds tremedous value to any code base.

### Why?

First, it serves as API documentation for the developers. Let's imagine the following JS code snippet:

```js
export class User {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}

function getFullName(user) {
  return `${user.firstName} ${user.lastName}`;
}
```

Check the `getFullName` function. What is this `user` parameter? What are its properties and methods?
Here, we can guess that it is a `User` instance. However, if the class and the function live in distinct 
files or directories, this guess becomes questionable.

Exploring such a JS code base can be a nightmare, as you would need to search for calls to `getFullName` 
to try to figure out what parameters are passed to it. The type system makes understanding existing code
much easier:

```ts
function getFullName(user: User) {
  return `${user.firstName} ${user.lastName}`;
}
```

It is now pretty clear that the `user` argument is an instance of the `User` class.

The type system also prevents a whole class of bugs, where some part of the code uses something -
a variable, method, function, or parameter - the wrong way. In plain JS, such bugs must be
covered by unit tests. In TypeScript, the transpiler will complain about such a misuse if things are properly
typed. The type system can completely replace a whole category of unit tests.

Let's imagine the following code:
```typescript
export class User {
  constructor(public firstName: string, public lastName: string) {}
}

function getFullName(user: User) {
  return `${user.firstname} ${user.lastname}`;
}
```

This code has a bug. The `getFullName` function contains typos: it uses the `firstname` and `lastname`
properties, while the `User` class defines those properties using a capital `N`: `firstName` and
`lastName`.

Here, the error is easy to spot, as the class definition and the function sit side by side. However,
if they live in distinct files, such a bug can be hard to spot without either a unit test or the 
TypeScript transpiler. Relying on the type system for such things is much simpler and quicker than 
writing unit tests.

### Everything, really?

Yes. Everything. For examples:

* The route parameters passed to the `activate` method of a route component,
* The `@bindable` properties of a custom element,
* Events published through the `EventAggregator`,
* Objects fetched using the `fetch` client,
* Didn't I say everything?

By typing everything, you eliminate a whole category of bugs. Additionally, you add documentation
for other developers and your future self. Lastly, you give more informations to your tools
(auto-completers, linters, bundlers, etc.) so they can do a better job assisting you.

Those advantages largely outweight the additional cost of properly typing things, trust me.
Or don't, and try it for yourself. You'll see.

## Small apps over big apps

Whenever possible, it is best to write multiple small applications over writing one huge application.
A small code base is more manageable than a large one, and it's easier to have multiple small teams 
maintaining multiple small applications than one huge team working on a single huge code base.
Small applications can be released independently and at different paces, while releasing huge apps
require much more planification between team members and features, and such a process is much less
agile.

I didn't invent anything here; microservices have been around long enough now, and they are based on the
same principles.

## Bundle intelligently

If you can't keep your application small as suggested before, at least use bundles intelligently so your 
application loads faster.

For example, if your application's users are segregated in roles, you can group features 
in role-based bundles. This way, a given user would load only the bundle(s) he needs to do his job
based on its role(s).

## Organize by features

Aurelia [features](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/app-configuration-and-startup/6){:target="_blank"}
are extremely useful to organize your code.

For example, I often have a `validation` feature in my Aurelia applications. This feature handles
some simple tasks related to form validation:

* It loads the `aurelia-validation` plugin,
* It registers at least one `ValidationRenderer` in the DI container,
* It registers any custom validation rules needed across the application.

Features can also be used to group components, models, and services related to specific domains.

A chapter of [my book](https://www.packtpub.com/web-development/learning-aurelia){:target="_blank"} is 
dedicated to this subject.

## Decompose

A route component which fetches data from an HTTP endpoint and renders a complex view itself will
likely be hard to maintain, as it has too many responsibilities, and makes it hard - or even impossible -
to reuse parts of its behavior or UI elements.

Components should be **loosely coupled** and **highly cohesive**. They should be small and do a simple
task. Larger components should be composed of smaller ones.

A lot of Aurelia's features can be of great help in decomposing components and making them flexible
enough to be used in larger components: one-way and two-way data-binding, content
projection, and template injection are the cornerstones of well-taylored components.

## Separate UI & domain concerns

Similarly, keep UI artifacts and domain artifacts separated in your code base.

**UI artifacts** are:

* Components,
* Custom elements & attributes,
* Value converters,
* Binding behaviors,
* Templates.

**Domain artifacts** are:

* Domain models,
* Domain services,
* HTTP clients.

For example, route components should be built by composing UI elements and domain artifacts together.
If it fetches its own data from an HTTP endpoint, defines its own model for its data and has a complex
template to present this data to the user, it will likely be hard to maintain and its constituents
impossible to reuse. Additionally, domain intelligence will be lost in UI concerns.

However, if you define distinct domain models and HTTP services, isolated from UI components, then
aggregate them all in a route component, the component will be smaller, will only coordinate interactions
between its constituents, and your code will likely be more sane and easier to understand.

## Have fun!

I had a lot of fun it the last year working with Aurelia. I hope you do too!

Don't hesitate to let me know about your own experience with Aurelia by leaving a comment. ☺