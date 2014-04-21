---
layout: post
title: "Look Who's Talking"
tags: [Twilio, CaaS]
---
{% include JB/setup %}

![Look who's talking](/assets/images/posts/look-who-is-talking.jpg)

# Once Upon a Time

Have a seat, I will tell you a story.

Before talking about [Twilio](http://www.twilio.com/), I'm going to talk about my motivation to use it. It was for a support phone app, [Support Roulette](https://github.com/phstc/support-roulette). After all, we didn't use it exactly as is, although we used part of it.


Due to the easiness to implement an app able to answer a call, record a voice message, send and receive a text message, in other words, an app which talks, I got inspired to write this post and also to present a [Tech Talk](https://github.com/phstc/support-roulette/tree/master/slides) (Vim talk hehe) about it.

## Support Roulette

We needed a phone support 24x7 in case the website becomes unavailable and other possible urgent issues.

Our idea was to nominate a support developer weekly.

The [MVP](http://en.wikipedia.org/wiki/Minimum_viable_product) was to buy a cheap cellphone (its battery is infinite, on the other hand, it doesn't even have the [snake game](http://en.wikipedia.org/wiki/Snake_(video_game\))) and keep it with the  developer on duty. The idea was mainly to have a fixed phone number for our customer service call, in case of emergency.

### Problems with the cheap cellphone

Everybody already has a cellphone. Carrying another is annoying. Easy, very easy to forget.

### Solution

We decided to use a Communication as a Service (CaaS) to register a fixed phone number in Brazil, which receives a call and redirects it to the developer on duty cellphone.

### Voip

The standard Voip services that we evaluated, even though they allow to register a fixed phone number in Brazil and redirect the calls, don't allow to change the redirect target number via API. We would have to access their control panel weekly and configure the new redirect target number. As developers, we want to schedule the support agenda and have an app that will notify the new developer on duty and make the redirects. 

## Why Twilio?

* No Contracts
* No Up Front
* PAY AS YOU GO
* Simple and well documented API

### Other options

There are [other options](http://en.wikipedia.org/wiki/Twilio#Competitors) of CaaS, however not all of them offer fixed phone number in Brazil or simple and well documented API. Based on what we evaluated, [Plivio](http://www.plivo.com/) seemed to be the best competitor.

## Apps which talk

Given communication (voice and text) powers to an app, we can add many interesting features, such as:

* Alerts & Notifications
  * Product stock updates
  * Notify when a website is unavailable. It is kind of "easy" to implement a homemade [Pingdom](https://www.pingdom.com/) with [Monit](http://mmonit.com/monit/) + Twilio
* Promotions
  * Send promotion codes
  * Rewards "Send a text to … to win a ..."
* Reminders
* Surveys
* System administration
  * Retrieve system diagnostics
  * Execute remote commands
* Sales Automation
  * Order or consult products
* Bike Sampa
  * In São Paulo we have renting bike stations throughout the city. To release the bike we need to open the Bike Sampa app (iOS or Android), select the station and the bike then press "rent it". It's ok, but we need an iPhone or Android, the app and 3G. It would be much easier to send a text with the station and the bike instead of the whole app flow (iOS or Android, app and 3G).
* Identity Verification
  * Validate user by phone
* Support Roulette ;)

Have look at [Twilio Customer Success Stories](http://www.twilio.com/gallery/customers) for other use cases.

### Homemade Pingdom with Monit + Twilio

Monit is an utility for managing and monitoring process, programs etc.

Pingdom is a service for monitoring uptime, downtime and performance of websites.

An usual usage of Pingdom is to use it only to monitor uptime, basically to notify someone if the website is down. We can achieve it in a homemade way with Monit + Twilio.

#### Monit script

    # /etc/monitrc
    
    check host rexnuke with address 127.0.0.1
      if failed port 8080
       with timeout 15 seconds
       then
       exec "ruby notify-site-is-down.rb"

#### Ruby script

    # /usr/bin/notify-site-is-down.rb
    require "rubygems"
    require "twilio-ruby"
    
    account_sid = "ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    auth_token = "yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy"
    
    @client = Twilio::REST::Client.new account_sid, auth_token
    
    @client.account.sms.messages.create(
      from: "+5511…",
      to: "+5511…",
      body: "Houston we have a problem, the website is down."
    )

### Costs

In the case of Support Roulette, there are two costs: one for voice, when Twilio receives a call and redirects it and another for text, to notify the new developer on duty ("Congratulations, you are the new developer on duty :trollface:").

#### Voice

The voice cost is in two legs: to receive and redirect (make) a call.

To receive a call: 1¢/minute

To make a call: 33¢/minute

#### Text

> SMS messaging from an American Twilio phone number to Brazil starts at 1.2¢ however this price will vary depending on the carrier you are sending SMS messages to.
> Elaine Tsai, Twilio Support

Brazilian fixed number phone lines don't send text yet. To send a text to Brazil, an American number is required. The main inconvenience of doing it, is, in case it's needed to send or reply a message, it must be to an American number.

To send a text: starts at 1.2¢/message

#### Fixed number

Fixed brazilian number: 3$/month

Fixed american number: 1$/month

## Getting started

The initial steps are: [TwiMLTM: the Twilio Markup Language](http://www.twilio.com/docs/api/twiml) and
[Twilio REST Web Service Interface](http://www.twilio.com/docs/api/rest).

Example of TwiMLTM:

    <?xml version="1.0" encoding="UTF-8"?>
    <Response>
        <Say voice="woman">Please leave a message after the tone.</Say>
    </Response>

I also recommend you to have a look at Support Roulete [source code](https://github.com/phstc/support-roulette), it is a simple [Sinatra](https://github.com/sinatra/sinatra) application which uses the [gem twilio-ruby](https://github.com/twilio/twilio-ruby) and [builders](https://github.com/phstc/support-roulette/blob/master/views/support_roulette_call.builder) to generate TwiMLTM. It can be free hosted on Heroku.

## Twimlets

[Twimlets](https://www.twilio.com/labs/twimlets) are tiny web applications that implement basic communication functionality, such as: [call forward](https://www.twilio.com/labs/twimlets/forward), [voicemail](https://www.twilio.com/labs/twimlets/voicemail) etc. 

If you are in New York but you want a fixed number in Rio de Janeiro, you can easily create a Twilio account, configure a twimlet for a forward call and in less than 10 minutes everything will be working as expected. Awesome!
