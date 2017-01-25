---
layout: post
title: "Hard-coded IDs suck, and enums won't help"
tags: [CSharp, Design]
published : true
--- 

Sometimes an application requires that some features are enabled (or rules applied), or not, based on some context. What is the best way to
solve such a problem?

## Hard-coded IDs... Hard-coded IDs everywhere...

A C# app I was working on for a worldwide company was used in many countries. Of course, each country had specific features that were enabled 
and rules that were applied only for them. The previous programmers had used a very naive approach to solve this problem: they tested the user's 
country ID:

{% highlight csharp %}
if (user.Country.ID == 1)
{
    // Show a button available only for country #1
    // By the way, country #1 is France for the sake of this example
}
{% endhighlight %}

The app ended up being full of those ugly checks. Plus, if you don't know the countries and their IDs and don't have the database table under hand, 
good luck trying to figure out what users of which country should see this button.

## Enums to the rescue - or not

I was absolutely not convinced by the first idea that came to my mind, but I decided to give it a try. I created an enum listing the countries:

{% highlight csharp %}
public enum CountryEnum
{
    France = 1,
    Canada = 2,
    // and so on
}
{% endhighlight %}

Then I added a method to the Country entity:

{% highlight csharp %}
public class Country
{
    // snip...

    public CountryEnum ToEnum()
    {
        return (CountryEnum)ID;
    }

    // snip...
}
{% endhighlight %}

And the checks turned into:


{% highlight csharp %}
if (user.Country.ToEnum() == CountryEnum.France)
{
    // Show a button available only for France
}
{% endhighlight %}

Well... it was a little better. It was much easier to see what features and rules applied to which country. But it still sucked. Mainly 
because it was hard to change. Imagine having a rule for France that would require to show or enable multiple buttons, links, table columns 
or whatever on multiple views. And imagine that Canada wants that same feature in the next release. It was still a huge "oh God no" for me to 
go and update all those checks to include those pesky Canadians. Add to that the fact that the pages and user controls (yes, it was a web forms app) 
were bloated with those checks, sometimes many of them entangled together, and you've got a huge mess on your hands. Time to sit back and think.

## Feature toggles

I remembered reading a while ago about [feature toggles](https://martinfowler.com/bliki/FeatureToggle.html) on Martin Fowler's bliki, and I 
realized that this solution, although brought up in a static context (the way Fowler describes them, feature toggles are enabled or disabled 
generally in configuration - in our example, each country would have its own live version of the app, configured for its own needs), could be 
applied in a dynamic context like mine.

So I rolled all my changes back, and started from scratch:

{% highlight csharp %}
public class CountryFeatures
{
    public bool CanDoThis { get; private set; }
    public bool CanSeeThat { get; private set; }
    public bool MustConfirmBeforeThatAction { get; private set; }
    // and so on for every country-specific feature or rule
}
{% endhighlight %}

You get the idea for the next step, I guess:

{% highlight csharp %}
public class Country
{
    // snip...

    public CountryFeatures Features { get; private set; }

    // snip...
}
{% endhighlight %}

How I initialized the Features property for each country is not relevant to this article: you could store the feature switches for each country 
in a database table, in a configuration file, or even hard-code them for all I care (although this solution kind of defeats the purpose of removing 
hard-coded country references - but not completely). What's important is that the feature checks were now much clearer:

{% highlight csharp %}
if (user.Country.Features.CanDoThis)
{
    // Show the button used to do this
}
{% endhighlight %}

Now that is a much better solution. The single responsibility principle is respected: the view only needs to know if a given feature is enabled 
or not to decide if it must render the button. It doesn't need to know for which country the feature is enabled anymore.

Additionally, even if this check was found in multiple places, there was no need to change all those occurrences anymore to enable the feature 
for Canada. I only needed to toggle the feature on for the countries that needed it.

## Summary

Feature toggle is a powerful tool that can be leveraged not only statically, through configuration, to enable or disable features on a per-deployment 
basis, but it can also be used dynamically, using some context information in a single deployment setup.