---
layout: post
author : Manuel Guilbault
category: Article
title: Truly embracing the OO paradigm
tagline: 
tags: [design]
published : false
--- 

It seems to me that many developers don't completely understand the object oriented paradigm. I've seen my share of model objects containing 
nothing by properties. Those kinds of design generally delegate all responsibilities to "controller", "manager" or "handler" objects. The 
model objects themselves have no responsibilities, other than being degenerated data structures.

## Context

Let's look at an example:

{% highlight csharp %}
public class ValueEntry
{
    public DateTime EffectiveFrom { get; set; }
    public DateTime? EffectiveUntil { get; set; }
    public decimal Amount { get; set; }
}

public class Value
{
    public Value()
    {
        Entries = new List<ValueEntry>();
    }

    public List<ValueEntry> Entries { get; set; }
}

public class ValueController
{
    public ValueEntry GetCurrentValue(Value value)
    {
        return value.Entries
            .FirstOrDefault(e => e.EffectiveUntil == null);
    }

    public void UpdateValue(Value value, decimal amount, 
        DateTime effectiveFrom)
    {
        var currentValue = GetCurrentValue(value);
        if (currentValue != null)
        {
            currentValue.EffectiveUntil = effectiveFrom;
        }

        value.Entries.Add(new ValueEntry()
        {
            EffectiveFrom = effectiveFrom,
            Amount = amount
        });
    }
}
{% endhighlight %}

The domain requirements for this example are pretty simple. Let's keep the rules related to the subject at hand (other rules, related to 
model integrity, will be the subject of another article):

1. A value contains the history of its past values, along with its current value.
2. Even though the EffectiveUntil property is redundant (it is equal to the EffectiveFrom property of the next value in history), the developers 
   chose to add it anyway in order to simplify some query operations. This forces the update operation to close the previous current value by 
   setting its EffectiveUntil property to the value of EffectiveFrom property of the new current value.

The problem with this approach is that it completely violates the spirit of the object-oriented paradigm. Those kinds of design are not 
object-oriented designs. An object is all about data and behavior. Here, data and behavior are completely separated, which makes the 
design less cohesive.

### Let's clarify something

I'm not saying that behavior must never be separated from data. In fact, some of the most useful design patterns (like the 
[strategy pattern](http://www.oodesign.com/strategy-pattern.html)) rely on it. What I'm actually saying is that this separation must be 
avoided when it is not useful.

## Refactoring

In this particular case, the first step is pretty simple: get rid of the ValueController and put the methods directly in Value:

{% highlight csharp %}
public class ValueEntry
{
    public DateTime EffectiveFrom { get; set; }
    public DateTime? EffectiveUntil { get; set; }
    public decimal Amount { get; set; }
}

public class Value
{
    public Value()
    {
        Entries = new List<ValueEntry>();
    }

    public List<ValueEntry> Entries { get; set; }

    public ValueEntry GetCurrentValue()
    {
        return Entries
            .FirstOrDefault(e => e.EffectiveUntil == null);
    }

    public void Update(decimal amount, DateTime effectiveFrom)
    {
        var currentValue = GetCurrentValue();
        if (currentValue != null)
        {
            currentValue.EffectiveUntil = effectiveFrom;
        }

        Entries.Add(new ValueEntry()
        {
            EffectiveFrom = effectiveFrom,
            Amount = amount
        });
    }
}
{% endhighlight %}

It looks a lot better now, with data and behavior reconciled inside the same model object. It the next article, we'll look at protecting invariants.
