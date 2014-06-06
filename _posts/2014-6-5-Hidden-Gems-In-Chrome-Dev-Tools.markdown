---
layout: post
title:  "Hidden Gems In Chrome Dev Tools"
date:   2014-6-5 8:00:00
categories: tools
---

I recently have had to do a bunch of performance benchmarking and like all developers, a fair deal of debugging. During this time [I ran across](https://www.youtube.com/watch?v=rcjUR4icvoQ) some hidden tools in the Chrome that are not officially documented any where by Google. Below is a break down of those tools.

##debug() and undebug()

If you ever had to manually setup a `debugger;` statement at the top of your function to pause when the function is called, you can avoid doing this by using `debug()`. This method is available in the console of the dev tools and accepts a function reference as an argument.

```js

var Person = {
    greet: function ( greeting ) {
        var name = 'Chad';
        console.log( greeting + ' ' + name );
    };
};

dubug( Person.greet );

Person.greet( 'Hi' ) // Will Pause
```
`debug()`'s counterpart is `undebug` that simply takes a reference to the function that was "debugged" and removes the debugger from it.

##monitor()

This one will save you from all of those times where you setup something like the following.

```js
var Person = {
    greet: function ( greeting ) {
        var name = 'Chad';
        console.log( 'greet called with ' + greeting );
    };
};
```

Basically `monitor()` is `debug()`'s more passive sibling.  When you use `montior()` the console will print out that the function was called and with the arguments that it was called with.

```js
var Person = {
    greet: function ( greeting ) {
        var name = 'Chad';
    };
};

monitor(Person.greet);

Person.greet('Hola') // Logs Person.greet called with arguments: Hola
```
`monitor()`'s counterpart is `unmonitor` that simply takes a reference to the function that was "monitored" and removes the monitor from it.

##monitorEvents() and unmonitorEvents()

If you ever had to `console.log` streams of events say `keydown`, you can simplify things by just using `monitorEvents`.  This is a documented API but it's pretty awesome.

```
monitorEvents(window, 'resize');

monitorEvents($('#message'), ['keyup', 'keydown']);

monitorEvents($('#box'), 'touch');
```

The first example will console all window resize events. The second uses the `$` console API to query the element then log any time a keyup or keydown event occurs. Finally the last one queries the item and then logs anytime any of the touch events occur.  There is a whole list of superset and subset of events the `monitorEvents` API covers.

Like all of the API's above, `monitorEvents` has an `unmonitorEvents` to do exactly what you would expect.

##getEventListeners()

This is another documented goodie that doesn't get much praise.  If you pass `getEventListeners` a node it will do a reverse lookup of all the events attached to that element.

```js
var bar = document.querySelector( '#bar' );

getEventListeners( bar ); 
// [ { click: [ { listener: someFn, remove: someFn } ] }]
```

This can save you a ton a time where you don't have a strutured way of setting up event listeners and you're getting some weird side effects because someone accidentially attached a listerner.

## Wrap Up 

In part 2 of this series we will look at the undocumented structural profiler and memory panes to allow us to analysis with extreme percesion what is going on in our code.



