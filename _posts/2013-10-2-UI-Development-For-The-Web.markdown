---
layout: post
title:  "UI Development For The Web"
date:   2013-10-2 10:18:00
categories: application architecture, javascript
---

For about the past 3 years I’ve done a lot research and development in the realm of frontend application architecture with a strong focus on JavaScript. As more and more logic moves to the client it will become more and more important to have complete architectures in place. In more recent times, when people here the words “JavaScript Architecture” they automatically think of Model-View-Controller(MVC). In it’s purest form MVC is not a complete architecture and it is more of pattern. The reason why it’s not a complete architecture is that it does not prescribe a solution for “every thing” you would want to do in a system. Lets look how GUI development has evolved in the past 40 years or so.

## 40 Years In The Making

In the early days of GUI development UI widgets were comprised of both presentation logic and domain logic. However in the early 70s it was decided that this was probably a bad idea and extremely limiting in terms of composited UI. So the concept of separated presentation and observation synchronization came into light. In separated presentation your domain logic layer notifies your presentation layer that then draws your UI. When an interaction occurs in UI that input is sent to the presentation layer which then notifies the domain logic layer.

![Separated Presentation](/assets/images/separated-presentation.png "Separated Presentation")

This pattern quickly evolved and Xerox Parc birthed MVC in Smalltalk-76. In MVC you still have the same type of separation but Xerox added the idea of a controller that would handle the raw input. In the Smalltalk-80 design the view and the controller are tightly coupled as they hold an instance variable of each other. This was done because complex observations begin to break down if they did not have references to one another. So essentially it’s a mechanism of code organization; anything that deals with drawing or updating the UI goes into a view, anything dealing with raw events goes into the controller. This is MVC in it’s purest form.

![Model-View-Controller](/assets/images/mvc.png "Model-View-Controller")

In the late 80s some SmallTalk developers spun off to form ParcPlace Systems. They ended up producing a cross-platform implementation of SmallTalk called VisualWorks. VisualWorks built on top of the classic SmallTalk-80 MVC pattern and introduced what they called the presentation model. The role of the presentation model was to sit in between the domain model and the view to keep track of application state that would not be persisted.

![Model-View-Controller with presentation model](/assets/images/mvc-presentation-model.png "Model-View-Controller with presentation model")

In the late 80s and early 90s, NeXT, which would be later acquired by Apple in 1996, made 3 changes to the MVC pattern making it a much more complete architecture. They had renamed the presentation model to the mediating controller, they added a new construct call the coordinating controller, and now the view would be responsible for drawing and raw input. The role of the coordinating controller is to coordinate communication of state and actions in a application composition hierarchy. In other words it is the object that is responsible of notifying other parts of the UI that something interesting has happened. It is also responsible for bootstrapping the objects for that section of the UI. It’s important to note that each level of the hierarchy knows more and more about the application state.

![Coaco Framework MVC](/assets/images/coaco-mvc.png "Coaco Framework MVC")

So these are the some of the most prescribed solutions on how to do GUI development. It’s 40 years of smart people trying to get GUI development down to a science and I think there is a lot to learn from it.

## Developing For The Web

In web development we basically have the culmination of 40 years of designs running at various levels across the web. We have globs of JavaScript that do not care about separation of concerns. We have separation of presentation with pure Backbone. We have many implementations of the classic SmallTalk-80 MVC but in JavaScript. We have tons of people rolling there own bastardized solutions to problems that have been solved long ago.

So while in the web community we think this MVC thing is a new frontier, it is not and we should really stop trying to prescribe completely new architectures or systems. Instead we should build upon what history has already sorted out for us.

An excellent example of this is Ember. The Ember project has basically taken the Apple/NeXT MVC architecture and ported it to JavaScript and at the same time adapting it for the web. At a high level pattern they are essentially the same thing.

![Ember.js MVC](/assets/images/ember-mvc.png "Ember.js MVC")

They had to adapt the architecture because we are dealing with a UI that has a URL. In a sense the URL is highest level of abstraction to a Web UI so it is fitting that the concept of a URL route takes the place of coordinating controller. They also just simplified the naming from mediating controller to just controller.

##The Road Forward
I truly believe that a lot of our problems are already solved for us when it comes to UI development for the web. As I mentioned above, more and more logic is going move to the client and I think time has shown us how to deal with these problems in a very sane way. We should not turn our noses up and say “I’m going to do it better” because it’s likely that you’re not going to be successful or you will find yourself rehashing ideas from the past.

##### Resources:

[http://martinfowler.com/eaaDev/uiArchs.html](http://martinfowler.com/eaaDev/uiArchs.html)

[http://st-www.cs.illinois.edu/users/smarch/st-docs/mvc.html](http://st-www.cs.illinois.edu/users/smarch/st-docs/mvc.html)

[http://en.wikipedia.org/wiki/History_of_the_graphical_user_interface#Augmentation_of_Human_Intellect_.28NLS.29](http://en.wikipedia.org/wiki/History_of_the_graphical_user_interface#Augmentation_of_Human_Intellect_.28NLS.29)

[http://en.wikipedia.org/wiki/VisualWorks](http://en.wikipedia.org/wiki/VisualWorks)

[http://en.wikipedia.org/wiki/Cocoa_(API)](http://en.wikipedia.org/wiki/Cocoa_(API))

[http://en.wikipedia.org/wiki/Objective-C](http://en.wikipedia.org/wiki/Objective-C)

[http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)

[https://developer.apple.com/library/mac/documentation/general/conceptual/devpedia-cocoacore/ControllerObject.html](https://developer.apple.com/library/mac/documentation/general/conceptual/devpedia-cocoacore/ControllerObject.html)

[http://emberjs.com/](http://emberjs.com/)