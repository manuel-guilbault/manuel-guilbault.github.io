---
layout: post
title: "What is Aurelia's .two-way binding command for?"
tags: [Aurelia, JS, TypeScript, Web]
image:
  feature: "aurelia-main-logo.svg"
  teaser: "aurelia-main-logo.svg"
published : false
--- 

One of [my book](https://www.packtpub.com/web-development/learning-aurelia)'s reader sent me an email earlier this week.
He was confused by a statement from the book and thought it might contain a typo or some kind of error. This statement
said that the `.two-way` binding command didn't adapt to its context, while `.bind` did.

I answered that it was no error, that `.bind` uses two-way binding when the target property supports it and falls
back to one-way binding when it doesn't, while `.two-way` forces two-way binding - although it may not be as clear as 
I intended, that's what I meant when I wrote that it didn't adapt to its context.

That question made me realize that at first glance, `.two-way` seems pretty useless. Why would someone want to force
two-way binding on a property that doesn't support it? It makes no sense.

It actually does, because in fact the `.bind` command uses the target property's *default binding mode*. By default,
for all native HTML elements' attributes and properties, Aurelia considers that the default binding mode is one-way, 
except for properties affected by user inputs on form-related elements, such as an `input`'s or a  `select`'s `value`.

However, when creating custom elements and attributes, you have complete control over the default binding mode of
your component's properties. One could easily imagine a custom attribute - or a custom element's property - whose 
default binding mode is one-way, but who supports two-way binding. In such a case, using the `.two-way` command would 
be the only way to force two-way binding, since the `.bind` command would use one-way binding by default.

It is likely that you'll never use `.two-way`, but it can still be useful in such a scenario.

Thanks to Christian for his question.