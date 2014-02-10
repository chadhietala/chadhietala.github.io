---
layout: post
title:  "Testing Ember Apps With Venus"
date:   2014-2-9 5:38:00
categories: emberjs
---
When I wrote my first Angular application 2 years ago I was blown away at it's awesome testing story. Although the whole dependency injection thing was a little weird at first, it proved to be one of the frameworks largest selling points. Ember on the other hand did not focus on a super solid testing story at first, but I would argue that Ember now has a better testing story than Angular. Not only does Ember have a dependency injection through the [container](http://www.slideshare.net/mixonic/containers-di), it also comes along with [Ember Testing](http://emberjs.com/guides/testing/integration/), which is a suite of tools for integration testing.

## Testing At LinkedIn

At LinkedIn we use this test runner [Venus](http://www.venusjs.org/), which allows us to very quickly run tests using any testing framework across a multitude of environments, including things like [Selenium Grid](http://docs.seleniumhq.org/docs/07_selenium_grid.jsp) and [Sauce Labs](https://saucelabs.com/). Generally speaking frontend developers are only responsible for writing unit tests for any javascript that is committed. This is regulated by a pre-commit hook.

Beyond unit testing we use [Capybara](https://github.com/jnicklas/capybara) for all of our integration testing.  Prior to releases a regression is preformed on the service that is about to be deployed. These regressions testing normally get scheduled as an overnight job by our QA counter parts. It's extremely time consuming and energy intensive.

That all being said, a lot of new products or refreshes that we have recently released have been JavaScript MVC applications. This poses new challenges in integration testing but has a large upside in terms of time to results and energy consumed in testing.

## A Word About Venus

As I mentioned above we use Venus to write all of our unit tests. The largest advantage of Venus is that configuring is super easy and super flexible. It achieves this through annotations at the top of your test file. Some examples of this are `@venus-library`, `@venus-include`, `@venus-include-group`,  and `@venus-fixture`. These just tell Venus what testing framework you want to use, what files to include in the test and if you need any specific fixture html for the test. The `@venus-include-group` maps to an object of files that you can place in your `.venusconfig` file.

Once you have your annotations setup you just run the `venus run -t path/to/test/test.spec.js` command from your terminal. This will spin up a server and address you can view your tests. You can also run `venus run -t path/to/test/test.spec.js -e ghost` to just run the tests through phantom.js. For more information about running your tests across different environments and other configurations, [check out the documentation](https://venusjs.readthedocs.org/en/latest/tutorials/environments.html#selenium-grid).

## Ember Testing

The development version of Ember ships with a package called [Ember Testing](http://emberjs.com/guides/testing/integration/). This suite of tools is essentially what Capybara brings to the table, but without the assertions and it's not written in Ruby. This is what a test looks like when using Ember Testing with [mocha](http://visionmedia.github.io/mocha/).

{% highlight javascript %}

/**
 * @venus-libray mocha
 * @venus-include-group ember
 * @venus-include ../ember-mocha-adapter.js
 * @venus-include #PROJECT/app.js
 */

App.rootElement = '#test-host';
App.setupForTesting();
App.injectTestHelpers();

describe( 'App Integration Tests', function () {

  beforeEach(function () {
    App.reset();
  });

  describe( '/', function () {

    it( 'should say "Welcome To Ember"', function () {
      visit('/');
      andThen( function () {
        expect( find( 'h1' ).text() ).to.be( 'Welcome To Ember' );
      });
    });

  });

});

{% endhighlight %}

As you can see, we have the Venus annotations at the top that tells Venus what to load for the test. You'll notice that we are not using mocha's `done()` method call even though this is an async test. This is because the adapter manages ansyc operations through a global promise. By default Ember uses a Qunit adapter, but the API allows you to write your own. In this case I'm using Teddy Zenny's [Ember Mocha Adapter](https://github.com/teddyzeenny/ember-mocha-adapter).

By making the tests promise based, we don't have to do stuff like `wait(5)` to wait 5 seconds to ensure that a response has come back. Take that time and multiple it over thousands of tests and you have some serious time and energy savings. Like I mentioned these methods are similar to Capybara. So `visit('/')` goes to the index route in your application. The `andThen()` function is actually async although it looks synchronous. It does this by latching onto a global promise object that the adapter creates if it exists. The `find()` call looks inside of the `App.rootElement` and is essentially jQuery's `.find()` method. Ember testing ships with `vist()`, `find()`, `fillIn()`, `click()`, and `keyEvent()`, but Ember provides an easy way to [write your own async helpers](http://emberjs.com/guides/testing/integration/#toc_creating-your-own-test-helpers)

The other interesting parts here are `App.rootElement`, `App.setupForTesting()`, `App.injectTestHelpers`, and `App.reset` in the `beforeEach()` hook.

<dl>
  <dt><strong>App.rootElement</strong></dt>
  <dd>Allows you to tell Ember where you want to inject the App. As mentioned before, this is also the context that the <code>find()</code> function will use.</dd>
  <dt><strong>App.setupForTesting</strong></dt>
  <dd>Defers the readiness of the application by setting the router's location to <code>'none'</code>. On <code>visit()</code> of a route the application's "readiness" is advanced and then passed url is set.</dd>
  <dt><strong>App.injectTestHelpers</strong></dt>
  <dd>All this does is inject the test helpers into the global scope.</dd>
  <dt><strong>App.reset</strong></dt>
  <dd>This will blow away all the objects in your application and bring you back to the index route.</dd>
</dl>

So this gives you an idea of the types of things you can do with Ember Testing.

## Unit Testing With Ember

Ember Testing is awesome, but what about unit tests? A simple unit test looks something like this.

**Implementation**
{% highlight javascript %}
App.Person = Ember.Object.extend({
  firstName: 'Chad',
  lastName: 'Hietala',
  fullName: function () {
    return this.get( 'firstName' ) + ' ' + this.get( 'lastName' );
  }.property( 'firstName', 'lastName' )
});
{% endhighlight %}

**Test**
{% highlight javascript %}
/**
 * @venus-libray mocha
 * @venus-include-group ember
 * @venus-include #PROJECT/app.js
 */

describe( 'Person', function () {
  var ctrl;
  beforeEach(function () {
    ctrl = App.Person.create();
  });

  it( 'should create a full name', function () {
    expect( ctrl.get( 'fullName' ) ).to.be( 'Chad Hietala' );
  });

});
{% endhighlight %}

This example is pretty straight forward example. I have a `Person` object that has `firstName` and `lastName` on it. I want to make sure that when a make a get call to `fullName` that the computed property returns the concatenated version of `firstName` and `lastName`.

What happens when your controller has a dependency on another controller through the `needs` property? The `needs` property is basically is sugared syntax for asking the container for a specific controller instance in memory. What is the container you may ask? Ember registers modules into an [inversion of control container](http://martinfowler.com/articles/injection.html) to inject dependencies into other objects. This allows for the modules to depend on other modules without coupling it. Since the instances are not directly on the object, rather injected at runtime, we can mock these cross module dependencies very easily. Let's look at an example on how we would test this.

**Implementation**
{% highlight javascript %}
App.BirdController = Ember.Controller.extend({
  needs: [ 'nest' ],
  nest: Ember.computed.alias( 'controllers.nest' )
  actions: {
    flyHome: function ( track ) {
      var nest = this.get( 'nest' );
      nest.arrive();
    }
  }
});

App.NestController = Ember.Controller.extend({
  arrive: function () {
    this.set( 'nestOccupied', true );
  }
});
{% endhighlight %}

**Test**
{% highlight javascript %}
/**
 * @venus-libray mocha
 * @venus-include-group ember
 * @venus-include #PROJECT/app.js
 */

describe( 'BirdController', function () {
  var container, bird, nest, nestSpy;

  beforeEach(function () {
    container = new Ember.Container();
    nestSpy = sinon.spy();
    container.register( 'controller:bird', App.BirdController );
    container.register( 'controller:nest', Ember.Object.create({
      arrive: incrementSpy
    }));
    bird = container.lookup( 'controller:bird' );
    nest = container.lookup( 'controller:nest' );
  });

  afterEach(function () {
    Ember.run( container, 'destroy' );
  });

  it( 'arrive at home', function () {
    bird.send( 'flyHome' );
    expect( nestSpy.callCount ).to.be( 1 );
  });

});
{% endhighlight %}

So here we have a `BirdController` that needs access to the `NestController` to let others know that the nest is occupied. When an inversion of control container is used, an instance of the container in which the module came from is place on that module. This is so that the module can ask the container to inject other objects which preside in the container on to the asking object. In our test we can mock just that by creating our own container and registering objects to it.

In the example we create a new `Ember.Container` then register our `BirdController` and a mock `Ember.Object` to the container. We then pull both out of the container and assign them to `bird` and `nest` respectively. At the point where we ask the container for the specific controller types, they are now containerized, meaning the `needs` call of our `BirdController` will look into our testing container and resolve to our `NestController` mock. We can then right our test as expected and we can assert that the arrive method is indeed being called.

## Wrap Up

So hopefully I shined so light on how you can pretty much do unit and integration testing very easily and very quickly, as opposed to other ways of testing. I also hope that you learned a little bit on how objects get strung together in Ember through it's container.






