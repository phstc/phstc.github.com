---
layout: post
title: "Website monitoring. You will become addicted"
category: 
tags: 
- RUM
- StatsD
- Graphite
- Dashboard
- English posts
---
{% include JB/setup %}

This year (2012) I attended to a [ThoughtWorks Radar](http://www.thoughtworks.com/articles/technology-radar-march-2012) presentation. The guys from TW talked about radar topics, including Health Check Pages technique, which is really interesting and motived my self to create this post.

## Health Check Pages

> We have found adding simple health check pages to applications is incredibly useful. This allows people to quickly understand the health of an individual node. We often extend them to add metrics like the number of orders placed, error rates, or similar information. Using simple embedded web servers, even non-web based services can easily expose internal information over HTTP. By using microformats, these web pages can easily be scraped by other monitoring tools to become part of holistic monitoring.

Since then, I started to create dashboards in my websites. They aren't admins - I also create admins - but these dashboards can be apart of them. They are simple pages listing last orders, messages sent, counters etc.

These dashboards are addictive, I usually reload my browser many times per day to see the changes. Although the information is also available in the admin, it's much more attractive a page only showing the most important data.

## Real User Monitoring (RUM)

[RUM](http://en.wikipedia.org/wiki/Real_user_monitoring) is a technique to record user interactions, in our case, in a website.

It is similar to the concept of Health Check Pages mentioned before, maybe the same, but RUM is the buzzword. :)

Are you a hipster? You should use RUM.

I'm working for the biggest crafts marketplace in Latin America, and this means that we have high traffic. It is exciting to see graphics from our website interactions, as we have a huge number of data to feed our graphics.

We are using [StatsD](https://github.com/etsy/statsd/) and [Graphite](http://graphite.wikidot.com/) to monitor our website interactions. We monitor our logins (from Facebook, direct etc), searches, products visualization, orders completion, exceptions thrown etc, we monitor everything interesting for us.

These graphics are very useful. They help us:

* check if there is something broken. Once we broke our login with Facebook after a deploy, we quickly realized the 'login with Facebook' graphic falling and fixed it really fast. These graphics make us feel more confident to "continuous" deploy.

* check the traffic curves during the day.

* check if our last e-mail campaign increased the traffic.

...

I strongly recommend this article [Measure Anything, Measure Everything](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/).

## Make it nicer

It is nice to have an URL where everybody can access these graphics. But.... it is nicer to have a big television showing these graphics in the middle of the office. 

We did it, we have a computer, an Apple TV and a television. The computer has Chrome tabs opened for Graphite, Google Analytics Realtime and Hudson (green or red). We also use a Google Chrome extension to automatically switch the tabs in a specific interval. 

The Apple TV can be expensive, but it is really useful. We can switch from the computer to an iPad streaming showing the Olympics Games or a presentation etc.

## Reinventing the wheel?

Maybe you are thinking... "Why not Google Analytics (Standard and Realtime), New Relic...", we do use them, but they achieve different purposes... we are talking about simple pages to list our specific metrics in real time.

## Final thoughts

If it moves, you can track it. Be creative and you will become addicted. 