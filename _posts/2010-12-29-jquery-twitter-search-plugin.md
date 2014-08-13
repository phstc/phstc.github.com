---
layout: post
title: jQuery Twitter Search Plugin
---

It's a jQuery plugin only for tweets search. Where you need to implement the layout to display the tweets. Actually, it means that you are free to use your own layout. ;)
<!--more-->

Check out on GitHub to see the full documentation [github.com/phstc/jquery-twitter](https://github.com/phstc/jquery-twitter).

##Motivation

I worked in a project where I needed to display the last tweets messages containing reference to a website.

I found many jQuery Twitter Plugins, but most of them already worked with searching and displaying the tweets. In the project I mentioned the layout to display the tweets was specific. So I couldn't use the plugins’ layout.

##What’s the difference

The available plugins on the internet use an ugly syntax to search.

    $.twitter.search({
       from: 'pablocantero',
       text: 'hello+world',
       container: 'myDivId'
    });

Twitter Search Plugin is DSL based.

    var tweets = new Twitter.tweets();
    tweets.containing('hello')
       .and('world')
       .from('pablocantero')
       .limit(4)
       .order('recent')
       .all(function(data){
          alert('It’s a customized function to display the tweets');
       });

##Download it

The jQuery Twitter Search Plugin is available on GitHub [github.com/phstc/jquery-twitter](https://github.com/phstc/jquery-twitter).

##Example

[gist.github.com/758666](https://gist.github.com/758666)

##Do you want to improve this JavaScript?

The project is available on github as an open source project [github.com/phstc/jquery-twitter](https://github.com/phstc/jquery-twitter).

You’re welcome to make your contributions and send them as a pull request.

Thanks to [@rdgr](http://twitter.com/#!/rdgr).
