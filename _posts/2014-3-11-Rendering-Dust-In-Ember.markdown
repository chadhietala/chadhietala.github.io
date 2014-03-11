---
layout: post
title:  "Rendering Dust Templates In Ember"
date:   2014-3-11 8:00:00
categories: hacks
---

At LinkedIn we use [Dust](http://linkedin.github.io/dustjs/) pretty heavily as our means of standardizing and sharing UI patterns. We have built up a pretty large library of these re-usable templates and it's not realistic for us to migrate all of these templates if we were to try out Ember. So what does one do in a case like this? On the one hand you have the option of forking Ember and re-writing parts of `Ember.View` and any associated Handlebar helpers. On the other hand you can just wrap the other template languages lookup and render method in Handlebar's helper. Neither approaches are pretty, but I feel that wrapping option is much more of a sustainable and opens up the option of migration over time.

##Wrapping Dust In Handlebars

I was pleasently suprised that the code below [just works even with data binding](http://emberjs.jsbin.com/seroyoqa/4), assuming your templates have been pre-compiled.

{% highlight javascript %}
Ember.Handlebars.helper('tl', function(templateName, options) {
  var retVal = null;
  dust.render(templateName, options.hash, function (err,out) {
    retVal = new Handlebars.SafeString(out);
  });
  return retVal;
});
{% endhighlight %}

{% highlight jinja %}
{% raw %}
{{tl 'badge' degreeBinding=degree}}
{% endraw %}
{% endhighlight %}

So why is this generally frowned upon? Well, for one you have two templating libraries on the page and secondly you're re-rendering, re-painting, and potentially [thrashing the layout](http://wilsonpage.co.uk/preventing-layout-thrashing/) on every data change. While the latter does not sound like big of a deal, reaching into the DOM and re-rendering that template when any of the data changes is expensive and is why things like [metamorph.js](https://github.com/tomhuda/metamorph.js/) exists. The only thing that may be in our favor here is that `dust.render` is asynchronous.

Luckily we can reduce this performance hit by using Ember's `unbound` Handlebars helper to tell Ember that the values in template do not need to be bound. In a lot of our re-usable templates we don't really have a need for the data in the templates to be bound. In the cases where we it would be nice for the template to update, we will be subjected to the performance hit.

{% highlight html %}
{% raw %}
{{unbound tl 'badge' degreeBinding=degree}}
{% endraw %}
{% endhighlight %}

So is this a perfect solution? No, but I'm not expecting it to be. My other option is to make Ember aware of Dust and that solution does not scale as nicely as this solution. In the grand scheme of things, I believe this is an acceptable compromise instead of not using Ember at all.




