---
layout: post
title: isFollowing Bookmarklet - Who is following who?
---


Have you ever wanted to check if a twitter profile is following another twitter profile, this bookmarklet can help you.
<!--more-->
isFollowing Bookmarklet – it’s a very simple JavaScript created to check if screen name (b) is following screen name (a).

##How to use it?
1. Click and drag <a href="javascript:(function(){/*@author Pablo Cantero - http://pablocantero.com/blog/2010/12/20/isfollowing-bookmarklet-who-is-following-who*/var isFollowing = function(twitterScreenNameA, twitterScreenNameB){jQuery.getJSON('http://twitter.com/statuses/followers.json?screen_name=' + twitterScreenNameA + '&callback=?', function(data){for(var i = 0; i < data.length; i++){if(data[i].screen_name === twitterScreenNameB.replace('@', '')){alert(twitterScreenNameB + ' is following ' + twitterScreenNameA);return;}}alert(twitterScreenNameB + ' is not following ' + twitterScreenNameA);});};var isFollowingPrompt = function(){var twitterScreenNameA = window.prompt('input the twitter screen name (a)', jQuery('.screen-name').text());var twitterScreenNameB = window.prompt('input the twitter screen name (b)');if(twitterScreenNameA === null || twitterScreenNameB === null){alert('you must provide the screen names (a) and (b)');} else {isFollowing(twitterScreenNameA, twitterScreenNameB);isFollowing(twitterScreenNameB, twitterScreenNameA);}};if(typeof jQuery === 'undefined') {var jQueryScript = document.createElement('script');jQueryScript.setAttribute('src','http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.js');jQueryScript.setAttribute('type','text/javascript');jQueryScript.onreadystatechange= function () {if (this.readyState === 'complete' || this.readyState === 'loaded'){isFollowingPrompt();}};document.getElementsByTagName('head')[0].appendChild(jQueryScript);} else {isFollowingPrompt();}})()" title="it’s a very simple JavaScript created to check if screen name (b) is following screen name (a)">isFollowing</a> to your bookmark bar in the browser;.

2. Click in isFollowing in your bookmark bar;

3. Fill the screen name (a);

4. Fill the screen name (b).

##The JavaScript source code

[gist.github.com/744291](https://gist.github.com/744291)

##Do you want to improve this JavaScript?

The project is available on github as an open source project [gist.github.com/744291](https://gist.github.com/744291).

You’re welcome to make your contributions and send them as a pull request.
