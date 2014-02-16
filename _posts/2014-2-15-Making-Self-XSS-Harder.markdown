---
layout: post
title:  "Making Self XSS A Little Harder"
date:   2014-2-15 3:30:00
categories: security
---

So I was looking around on Facebook with my Chrome Developer tools open looking to see how Facebook was using [React](http://facebook.github.io/react/) in production ( or if they really were ), and I stumbled upon the fact that Facebook has disabled the console.

![Disabled Facebook Console](/assets/images/fb-console.png "Disabled Facebook Console")

I thought this was kind of interesting so I dug a little deeper on how and why this was done. What I found was a [stackoverflow post](http://stackoverflow.com/questions/21692646/how-does-facebook-disable-browsers-integrated-developer-tools/21692733) made by the security engineer at FaceBook and he explained why the disabled console was there.

> We're testing this for some users to see if it can slow down some attacks where users are tricked into pasting (malicious) javascript into the browser console.

So pretty much it's there so it's harder for users to commit [self XSS](https://www.facebook.com/photo.php?v=956977232793). Chrome wraps all of input from the console in the snippet below, so this provides us to disable the default behaviour of the console API.

{% highlight javascript %}
with ((window && window.console && window.console._commandLineAPI) || {}) {
  // Run The Codez
}
{% endhighlight %}

## Monkey Patch That Sucker
So the solution to make this style of self XSS a little bit harder is to monkey patch the accesors to `_commandLineAPI`. Either copy and paste the javascript into the console or checkout [this jsbin](http://jsbin.com/vuqam/1), open the console, and try to execute some code.

{% highlight javascript %}
(function(){
  var _c = console;
  _c.log('%c PROBLEM?', 'background: url(https://pbs.twimg.com/profile_images/378800000051679172/2076c82024cd9f2fb0b10d1f50258d63.png); font-size: 171px; text-align: center; color: #bada55; text-shadow: rgba(0,0,0, .5) 0 3px 3px;');
  Object.defineProperty( window, 'console', {
    get : function(){
      if( _c._commandLineAPI ){
        throw 'We stopped you from comitting self-xss.';
      }
      return _c;
    },
    set : function(val){
      _c = val;
    }
  });
})();
{% endhighlight %}

Pretty neat safety net.

