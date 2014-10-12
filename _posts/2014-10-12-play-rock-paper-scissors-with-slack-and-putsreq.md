---
layout: post
title: Play Rock-paper-scissors with Slack and PutsReq
---

Inspired by [Say Hello World to Wombat using PutsReq
](https://wombat.co/blog/say-hello-world-to-wombat-using-putsreq), this post will give you an introduction on [Slack Outgoing webhooks](https://my.slack.com/services/new/outgoing-webhook). 

Outgoing webhooks allows you to get data in and out of Slack in real time when someone send a message in a specific channel or with a trigger word.

## Fork Rock-paper-scissors webhook

First, let’s fork the Rock-paper-scissors webhook on PutsReq.

* Open [PutsReq Rock-paper-scissors](https://putsreq.herokuapp.com/hfep98jThPU0ffRlmWBx/inspect)
* Click on Fork

![](/assets/images/posts/slack-putsreq/putsreq-fork.png)

* Copy your PutsReq URL, it will be used as your webhook URL in the next step

![](/assets/images/posts/slack-putsreq/copy-putsreq-url.png)

## Add Rock-paper-scissors Outgoing webhook

* Access your [Slack Services](https://my.slack.com/services/new)
* Click on Outgoing WebHooks

![](/assets/images/posts/slack-putsreq/outgoing-webhooks.png)

* Click on Add Outgoing Webhook
* Fill Channel: Any, Trigger Word(s): rock, scissors, paper and URL(s): your PutsReq URL.

![](/assets/images/posts/slack-putsreq/integration-settings.png)

* Click on Save

## Play it

![](/assets/images/posts/slack-putsreq/play.png)

## More?

Now that your [Rock-paper-scissors](https://en.wikipedia.org/wiki/Rock-paper-scissors) is working, why not expand it with [Rock-paper-scissors-lizard-Spock](https://en.wikipedia.org/wiki/Rock-paper-scissors-lizard-Spock)? If you do that, please share! :)
