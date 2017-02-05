---
layout: post
title: "The computed properties problem"
tags: [CSharp, WPF]
published : false
--- 

I recently started working on a new Windows Phone 8 application at the office. It was pretty new to me, since most of my experience has been in a web-based environment.
I had never worked with WPF, yet I was pretty familiar with the MVVM pattern from working quite extensively with Knockout JS.

I was pretty annoyed when I realized that WPF has no support out of the box for computed values. I first started by wiring everything by hand. 
It turned out to look like this :

{% highlight csharp %}
public class ViewModel : BindableViewModel {
    private bool isLoading = false;
    pubic bool IsLoading
    {
        get { return isLoading; }
        set
        {
            SetProperty(ref isLoading, value);
            OnPropertyChanged("CanDoSomething");
        }
    }

    private bool isEnabled = true;
    public bool IsEnabled
    {
        get { return isEnabled; }
        set
        {
            SetProperty(ref isEnabled, value);
            OnPropertyChanged("CanDoSomething");
        }
    }

    public bool CanDoSomething
    {
        get { return !IsLoading && IsEnabled; }
    }
}
{% endhighlight %}

I thought this was pretty ugly, and that it could quickly grow into an incredible mess. Additionally, with this implementation, the PropertyChanged event 
for CanDoSomething could be fired even when the actual value of CanDoSomething didn't change. I decided to try and refactor the code to make the dependency 
between the properties a little more clear, and to fix the event being fired when no change occured issue. This is how it turned:

{% highlight csharp %}
public class ViewModel : BindableViewModel {
    private bool isLoading = false;
    pubic bool IsLoading
    {
        get { return isLoading; }
        set { SetProperty(ref isLoading, value); }
    }

    private bool isEnabled = true;
    public bool IsEnabled
    {
        get { return isEnabled; }
        set { SetProperty(ref isEnabled, value); }
    }

    private bool canDoSomething;
    public bool CanDoSomething
    {
        get { return canDoSomething; }
        private set { SetProperty(ref canDoSomething, value); }
    }

    public ViewModel()
    {
        PropertyChanged += (s, e) =>
        {
            switch (e.PropertyName)
            {
                case "IsLoading":
                case "IsEnabled":
                    CanDoSomething = !IsLoading && IsEnabled;
                    break;
            }
        };
    }
}
{% endhighlight %}

Yet, I still found this solution pretty complicated. It required a lot of code written. I had the feeling there could be a much more elegant 
(and reusable) solution for this.

## Solution 1

My first approach was to use expression trees. A class would wrap an expression, use this expression to compute a value, and analyze this expression 
to find all dependencies, so it can listen to them and be notified when a dependency changes, and then would re-evaluate the expression and fire an 
event if the computed value actually changed.