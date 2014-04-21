---
layout: post
title: "I coded a Backbone App in Rails and it took much less code"
tags: [SPA, Backbone, Rails]
---
{% include JB/setup %}

**tl;dr** single-page applications are awesome and fast. But if you don't have a strong requirement to create a single-page application, don't do that. It will slow down your development time. You can rewrite your application to be a single-page application in the future as an improvement, but if you start it as a single-page application and you need to deliver fast, you can get a big headache to yourself and maybe need to rewrite it as a normal web application (in our case in Rails) to meet the deadlines.

DISCLAIMER: Although the AngularJS first commit ([Jan 05, 2010](https://github.com/angular/angular.js/commit/c9c176a53b1632ca2b1c6ed27382ab72ac21d45d)) is older than the Backbone one ([Sep 30, 2010](https://github.com/jashkenas/backbone/commit/8a960b479859d343a6c734eb1a5817a2ff6c2b52)), for me, Backbone became a reference for MVW before AngularJS or any other, so that's why I'm referring to Backbone as the oldest one.

Another day a friend of mine shared a post with me - "[I Coded the AngularJS Tutorial App in Backbone and it Took 260% More Code](http://blog.42floors.com/coded-angular-tutorial-app-backbone-took-260-code)". It is an interesting post showing that newer MVW frameworks such as AngularJS require less code to do the work than the "old ones" like Backbone. This comparison reminded me a long time ago, in the Java world, the changes from HttpServlets (spaghetti servlets), then Struts (struts-config.xml), then Spring MVC (annotations) and so on. 

In my opinion it is the way to go. Backbone formalised a convention to write Rich Internet Application by creating "classes" with specific responsibilities, it uses Routes, Models, Collections, Views and Templates instead of spaghetti JavaScript with Ajax calls. But on the other hand you have to write all callbacks, bindings etc and it isn't magically implemented by a convention, like using data attributes (ng-*).

So, if you have a requirement to be a single-page application, I mean, if it must be a single-page application, go and check what's the best, maybe the "standard" isn't the best one anymore.

**If you don't have a strong requirement to create a single-page application, DON'T DO THAT!**

## Single-page applications can slow down your development time

### Ruby vs JavaScript

No matter how much you like JavaScript <3, calculate how many hours you spend coding in JavaScript and how many you spend coding in Ruby.

I really like JavaScript and I think I'm good at, but I usually spend many many more hours coding in Ruby than in JavaScript. So it isn't fair to compare my proficiency in Ruby which I'm coding all the time with JavaScript.

So, if you need to deliver fast, go with the language and tools you know more!

### Write everything twice

For a normal Customer CRUD on Rails, you usually have:
 
* Customer Model
* Customer Controller
* Customer views: index.html.erb, show.html.erb, new.html.erb, show.html.erb and _form.html.erb

For a normal Customer CRUD on Rails and Backbone, you usually have:

On Rails side:

* Customer Model
* Customer Controller
* Customer views: index.rabl, show.rabl

*[RABL](https://github.com/nesquena/rabl) is just an example, it could be with [ActiveModel::Serializers](https://github.com/rails-api/active_model_serializers) or whatever, but don't believe that you will be able to use only [Responders](http://api.rubyonrails.org/classes/ActionController/Responder.html) and models all the time.*

On Backbone side:

* Customer Collection
* Customer Model
* Customer View
* Customer templates: index.jst.eco, show.jst.eco, new.jst.eco, show.html.jst.eco and _form.jst.eco

You have much more code and you can't reuse Rails helpers. You can try to use [js-routes](https://github.com/railsware/js-routes), but you will probably find yourself trying to find some  JavaScript libraries analogue for [Rails TextHelpers](http://api.rubyonrails.org/classes/ActionView/Helpers/TextHelper.html).

[Kaminari](https://github.com/amatsuda/kaminari) for pagination,  no way, find something similar or write one by yourself.

### Everything will be an API

Sometimes in Rails we write some very specific actions i.e. `PUT users/:id/bio`, because there is a specific page, which only updates the user bio and has some specific logic. As it is something internal, we don't mind for the Restful like route, because it "isn't public exposed". But when we expose it as an API to access from Backbone, even though it isn't a real API, we can start overthinking on the "best fit" url, nouns and verbs etc. 

Sometimes it can be an anti-pattern, but who's never used a [helper_method](http://apidock.com/rails/ActionController/Helpers/ClassMethods/helper_method)? You can't use it anymore. If you want to, you will need to expose it as an API.

Backbone works "automatically" with Rails resources URLs, but for custom URLs, we can't easily create a form routing to the new specific URL. We need to create a specific method in the User model to make a request to the specific URL and a binding in the User view to call this specific method.

### Page reload is a user feedback

It can be a poor user feedback, but page reload is a user feedback. If a user clicks on the Save button and the page is reloaded, the user knows that something happened, even without an explicit feedback. But if the Save is handled by Ajax, you must do something: show an ajax loader, alert or whatever, otherwise it will be frustrating, the users will not be sure if the action was properly done and they will probably click on the button multiple times, just to make sure it was done, causing much more requests to your server.

### Page reload is a garbage collector

Do you clean your Backbone objects? I mean, if the user open a view which loads N objects (collection, models, sub-views, templates etc), paginate, load more N objects then triggers a new route which loads a new view and other objects. Do you erase the previous view and nested objects? If the user returns to the previous view, do you reuse the previous loaded view to keep the previous state? If yes, if the user opens 20 views with N objects each, do you keep all these objects in memory?

These are the questions you have to ask to yourself when developing a real single-page application. You have to take care   of the memory usage by hand - it isn't magically handled by Backbone. If you never used Chrome/Developer/Profile or similar, you should to!

You can get surprised on how a single-page application with many many views can becomes slow and consumes a lot of memory.

Page reload erases the page memory for almost every user action.

### Test

It can be subjective, but in my opinion RSpec and Capybara are much easier and complete than any other JavaScript test framework that I know.

For those who are already working on a single-page application, do you feel confidence with your JavaScript test coverage? Can you unit test a Backbone View manipulating DOM objects and feel proud on what we had to do to test it? I think many people can, but I couldn't.

## Are single-page applications good?

Yes, they are awesome. I love when I'm using Gmail and it doesn't reload the page and also keeps the previous state. I can do things without Internet connection, because they are loaded in the browser.

My point is just this: single-page applications frameworks/ecosystem aren't ready enough for everything; might be in the future, but for now it can be too expensive to develop and the users are more used to the UX of normal web applications than the single-page (desktop like) ones. You can't do the same UX for both. 

You can rewrite your application to be a single-page application in the future, the bad is when you need to rollback to a normal one. If you start in Rails, you can rewrite one page then another and so on, but if you start a project only with the JavaScript ecosystem outside Ruby/Rails (it was our case), it will be hard to migrate to Rails smoothly. 

Something I did before, which worked great, was to develop just specific parts of the application with Backbone (can be any other MVW). For example, we developed a Store Palette Editor using Backbone Models and Views, without Routers or Templates. It worked great, we could use the Backbone convention for Models and Views, isolate the classes with specific responsibilities, the UX was great and it made the development faster and the code easier to test.