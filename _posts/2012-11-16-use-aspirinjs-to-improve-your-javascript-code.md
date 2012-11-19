---
layout: post
title: Use AspirinJS to improve your JavaScript code
tags: [Javascript, Refactory, AspirinJS]
---
{% include JB/setup %}

Have you a JavaScript causing you a headache? Will you write a JavaScript that you don't want to cause you a headache?

AspirinJS is a collection of refactories to convert monolithic JavaScript code into maintainable, modular and testable JavaScript without any framework.

AspirinJS isn't a JavaScript Framework! It is only a few practices to be used as a reference to improve JavaScript code smoothly.

## Why?

Jeff Atwood wrote an interesting post arguing about how difficult it is to maintain a legacy code [When Understanding means Rewriting](http://www.codinghorror.com/blog/2006/09/when-understanding-means-rewriting.html). One of the most interesting points in this post is this graphic:

![When Understanding means Rewriting](http://codinghorror.typepad.com/.a/6a0120a85dcdae970b0120a86d7477970b-pi)

We spend more time understanding and editing code than writing new code. Writing a procedural Javascript with lot of callbacks is easy and cool when it is new, but when we need to maintain that code, it becomes too hard to understand and edit even if the code was written by ourselves.

> Any fool can write code that a computer can understand. Good programmers write code that humans can understand. [Martin Fowler, Refactoring: Improving the Design of Existing Code]

## Motivation

I do like [Backbone](http://backbonejs.org) and encourage its use and other Javascript frameworks as well, but sometimes it is too hard to add them to legacy code, e.g. projects which the resource urls don't follow any convention or no well defined resources etc.

JavaScript is a beautiful and sexy programming language, therefore we can write good code with pure JavaScript.

## Disclaimer

I'm using a little jQuery in AspirinJS - little means $.ajax, $.extend, event handler (trigger and on) and DOM manipulation. I used jQuery only to avoid boilerplate code - it isn't directly related to the refactory.

## Let's go, show me the code.

The full version of AspirinJS is available on GitHub [github.com/phstc/aspirinjs/](https://github.com/phstc/aspirinjs/). Here I will show only the before and after, on GitHub you can have look in each version and its own achievement.

### Before

**index.html**

    <html>
      <head>
        <title>Twitter search</title>
        <script src="//ajax.googleapis.com/ajax/libs/jquery/1.8.2/jquery.min.js"></script>
        <script>
          $(function(){
            $("#search-query").focus();

            $("#search-form").submit(function(event){
              event.preventDefault();
              $("#search-results").html("Loading...");
              var searchQuery = $("#search-query").val();
              $.ajax({
                url: "http://search.twitter.com/search.json?q=" + searchQuery,
                dataType: "jsonp"
                }).done(function(data){
                var resultsContainer = $("<ul></ul>");
                for(var i = 0; i < data.results.length; i++){
                  var result = data.results[i];
                  resultsContainer.append("<li>" + result.text + "</li>");
                }
                $("#search-results").html(resultsContainer);
              });
            });
          });
        </script>
      </head>
      <body>
        <h2>Twitter search</h2>
        <form id="search-form">
          <p><input type="text" id="search-query" placeholder="Search query"/>&nbsp;<input type="submit" value="Search"/></p>
        </form>
        <h2>Search results</h2>
        <div id="search-results"></div>
      </body>
    </html>

### After

**index.html**

    <html>
      <head>
        <title>Twitter search</title>
        <script src="//ajax.googleapis.com/ajax/libs/jquery/1.8.2/jquery.min.js"></script>
        <script src="assets/javascripts/app/application.js"></script>
      </head>
      <body>
        <h2>Twitter search</h2>
        <form id="search-form">
          <p><input type="text" id="search-query" placeholder="Search query"/>&nbsp;<input type="submit" value="Search"/></p>
        </form>
        <h2>Search results</h2>
        <div id="search-results"></div>
      </body>
    </html>

**assets/javascripts/app/models/data_model.coffee**

    class app.models.DataModel

      getData:  (url, data={}) ->
        $.ajax url, data

**assets/javascripts/app/models/twitter_model.coffee**

    class app.models.TwitterModel extends app.models.DataModel

      search: (query) ->
        url = "http://search.twitter.com/search.json?q=#{query}"
        this.getData(url, dataType: "jsonp").
          success (data) ->
            $(window).trigger "TwitterModel::search", data

**assets/javascripts/app/application.coffee**

    window.app =
      models: {}
      views: {}

    $ ->
      twitterView = new app.views.TwitterView
      twitterView.render()

**assets/javascripts/app/views/twitter_view.coffee**

    class app.views.TwitterView

      twitterModel = new app.models.TwitterModel

      render: ->
        $("#search-query").focus()
        $("#search-form").submit (event) =>
          event.preventDefault()
          @search()
        $(window).on("TwitterModel::search", @printSearchResults)

      search: ->
        $("#search-results").html "Loading..."
        searchQuery = $("#search-query").val()

        twitterModel.search searchQuery

      printSearchResults: (event, data) ->
        resultsContainer = $ "<ul></ul>"
        for result in data.results
          resultsContainer.append "<li>#{result.text}</li>"
        $("#search-results").html resultsContainer

    $ ->
      twitterView = new app.views.TwitterView
      twitterView.render()

**specs/javascripts/app/models/data_model_spec.coffee**

    require "//ajax.googleapis.com/ajax/libs/jquery/1.8.2/jquery.min.js"
    require "/application.js"
    require "/models/data_model.js"

    describe "Data model", ->

      it "gets data from an url", ->
        data_model = new app.models.DataModel
        spyOn $, "ajax"
        data_model.getData "localhost"
        expect($.ajax).toHaveBeenCalledWith "localhost", {}

**specs/javascripts/app/models/twitter_model_spec.coffee**

    require "//ajax.googleapis.com/ajax/libs/jquery/1.8.2/jquery.min.js"
    require "/application.js"

    describe "Twitter Model", ->

      it "searches tweets", ->
        twitter = new app.models.TwitterModel
        spyOn($, "ajax").andReturn success: ->
        twitter.search "vapormvc"
        expect($.ajax).toHaveBeenCalledWith "http://search.twitter.com/search.json?q=vapormvc", dataType:  "jsonp"

## Too much code?

Even if it seemed to much code for a basic example of "Tweet search", believe me, all code start simple and easy to understand, but as they are used, they grow. Modular code following [Single Responsibility Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle), will avoid a huge headache in the future.