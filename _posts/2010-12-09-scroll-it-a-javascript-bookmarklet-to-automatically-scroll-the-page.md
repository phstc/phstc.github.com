--- 
layout: post
title: Scroll it - A Javascript bookmarklet to automatically scroll the page
tags: 
- Bookmarklet
- English posts
- JavaScript
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _syntaxhighlighter_encoded: "1"
  dsq_thread_id: "187940788"
  _wp_old_slug: scroll-it-a-javascript-bookmarklet-to-make-automatic-scrolling-in-the-page
---

Have you ever had trouble when trying to play a song in your guitar when reading the chords from a web site? Don't you always need to stop playing to scroll the page?

Scroll It – it’s a very simple JavaScript created to solve this kind of problem! It makes automatic scrolls in the page in a very comfortable speed.

<!--more-->

##How to use it?

<del>
1. Click and drag Scroll it to your bookmark bar in the browser;
</del>

Unfortunately I needed to remove the Bookmarklet link. The Markdown compiler didn't worked with it.

2. Go to the site that your want to use Scroll it;

3. Click on Scroll It in your bookmark bar. That’s all.

To stop the automatic scroll, you only need to click again on Scroll it in your bookmark bar. ;)

##The JavaScript source code

    javascript:(function(){
    	/* @author Pablo Cantero http://pablocantero.com */
    	if(window.scrollIt === undefined){
    		window.scrollIt = {
    			intervalId: null
    		};
    	}
    	function getScrollMaxSize(){
    		return document.body.clientHeight;
    	};
    	function getScrollCurrentSize(){
    		return document.documentElement.scrollTop;
    	};
    	window.scrollIt.initialSize = getScrollCurrentSize();
    	window.scrollIt.scrolled = getScrollCurrentSize();
    	var scroll = function(){
    		window.scrollIt.scrolled += 10;
    		window.scrollTo(0, window.scrollIt.scrolled);
    		if(window.scrollIt.scrolled &amp;gt;= getScrollMaxSize()){
    			window.scrollIt.scrolled = window.scrollIt.initialSize;
    		}
    	};
    	var stop = function(){
    		window.clearInterval(window.scrollIt.intervalId);
    		window.scrollIt.intervalId = null;
    	};
    	var start = function(){
    		window.scrollIt.intervalId = window.setInterval(scroll, 1000);
    	};
    	if(window.scrollIt.intervalId != null){
    		stop();
    	} else {
    		start();
    	};
    })()

##Do you want to improve this JavaScript?

The project is available on github as an open source project.

[github.com/phstc/scroll-it](https://github.com/phstc/scroll-it)

You’re welcome to make your contributions and send them as a pull request.

Thanks to [@rdgr](http://twitter.com/#!/rdgr).
