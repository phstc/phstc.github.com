---
layout: post
title: Sidekiq (Redis) vs Shoryuken (AWS SQS)
---

Disclaimer: This post is not intended to compare [Sidekiq](http://sidekiq.org/) and [Shoryuken](https://github.com/phstc/shoryuken) implementations. Both are very similar, actually Shoryuken started as a [shameless copy of Sidekiq](https://github.com/phstc/shoryuken#credits) <3 open source. The idea of this post is to compare Sidekiq as a Redis based queue system with Shoryuken as a [SQS](https://aws.amazon.com/sqs/) client.


## Resque > Sidekiq > Shoryuken

Before Sidekiq, [Resque](https://github.com/resque/resque) was the main option to go for background jobs for many Rubyists. But Sidekiq changed that, mainly because it uses threads instead of process forking (Resque). No matter how many improvements MRI is receiving on process forking, we can't get the same performance (usage of resources) using forks as we get with threads. You can check some [Sidekiq testimonials](https://github.com/mperham/sidekiq/wiki/Testimonials) confirming that.

But the game is changing again - it's Shoryuken time.

With Sidekiq:

* If you are not using the [Pro version](http://sidekiq.org/pro/) (which costs $750/year) and your process crashes while consuming jobs, you will lose your jobs
* If your Redis master crashes before having replicated your jobs to the slaves, you will lose your jobs
* If your jobs load increases, you will need to have a more powerful Redis. More jobs, more Redis, more Money
* If you don't have a Redis cluster, eventually you will lose everything
* No Dead Letter Queues support
* No Fanout support
* You need to host its dashboard (console)
* You can use [sidekiq-statsd](https://github.com/phstc/sidekiq-statsd) or [Sidekiq pro/Metrics](https://github.com/mperham/sidekiq/wiki/Metrics) to send metrics to Statsd for distribution to Graphite, Librato Metrics, etc, but you will need these services running somewhere

With Shoryuken:

* If your process crashes, your jobs will return to the queue after its [visibility timeout](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) expiration, nothing will be lost
* If a SQS node crashes, you don't care, nothing will be lost
* If your jobs load increases from 1 to 1 million, you don't care, SQS handles that for you
* [Dead Letter Queues support](/blog/2014/11/29/sqs-to-the-rescue/#dead-letter-queues) - move messages after a configurable number of failures to a specific queue
* [Fanout support via SNS](/blog/2014/11/29/sqs-to-the-rescue/#sns-to-sqs) - send a message once and distribute across multiple queues
* [Hosted console](https://console.aws.amazon.com/sqs/home?region=us-east-1)
* You can [monitor your queues with CloudWatch](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/MonitorSQSwithCloudWatch.html#SQS_metricscollected) and send [alarm emails](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/AlarmThatSendsEmail.html) (SMS messages, HTTP calls) when a threshold is breached

## Perfomance

WARNING: SUBJECTIVE PERFORMANCE TESTS!!!

Are you wondering about performance? I would be, Redis is super fast.

To test performance, I created two projects:

* [https://github.com/phstc/sidekiq-putsreq](https://github.com/phstc/sidekiq-putsreq)
* [https://github.com/phstc/shoryuken-putsreq](https://github.com/phstc/shoryuken-putsreq)

Test plan:

* Send 1k jobs
* Make a HTTP call per job to a [PutsReq](http://putsreq.com/) bucket
* Compare the "From first to last" result

Result:

* Sidekiq took 13 seconds to consume 1k jobs
* Shoryuken took 14 seconds to consume 1k jobs

![](/assets/images/posts/sidekiq-putsreq.png)

![](/assets/images/posts/shoryuken-putsreq.png)

Although the jobs consumption result was pretty close, sending jobs with Sidekiq is clearly faster than with Shoryuken.

* Sidekiq took 2278.37ms to send 1k jobs (2.28ms per send)
* Shoryuken took 55629.93ms to send 1k jobs (55.63ms per send)

Yeah, Redis write performance is amazing. But is 55.63ms much?

*Every time you run these tests you will get a different result, but in most cases Sidekiq is slightly faster to consume and clearly faster to send jobs than Shoryuken.*

*More than once while trying bigger batches with Sidekiq, I got less requests than the batch size on PutsReq. Shoryuken was consistent, the number of jobs sent was the number of requests shown on PutsReq.*

## Pricing

> SQS is a paid solution. You can run Sidekiq for free using Heroku with something like Redis To Go.

Nope! SQS can be also free or cheaper than Sidekiq:

### Shoryuken

[Shoryuken pricing](http://github.com/phstc/shoryuken):

* $0


[SQS pricing](https://aws.amazon.com/sqs/pricing/):

* First 1 million Amazon SQS Requests per month are free
* $0.50 per 1 million Amazon SQS Requests  per month thereafter ($0.00000050 per SQS Request)

### Sidekiq

[Sidekiq pricing](http://sidekiq.org):

* $0

[Sidekiq Pro pricing](http://sidekiq.org/pro):

* $750/year

[Redis To Go pricing](https://addons.heroku.com/redistogo):

* Starts at $0 for a very limited option. But keep in mind that Redis To Go isn't an [HA solution](https://en.wikipedia.org/wiki/High_availability), they don't offer a Redis cluster, so it's a [SPOF](https://en.wikipedia.org/wiki/Single_point_of_failure). For an HA solution they recommend [ObjectRocket](https://objectrocket.com/pricing), which starts at $59/month

## From Sidekiq to Shoryuken

Did I convince you?

Have a look at the migration steps: [From Sidekiq to Shoryuken](https://github.com/phstc/shoryuken/wiki/From-Sidekiq-to-Shoryuken).

## Conclusion

There's no silver bullet solution. If your workers aren't thread safe workers, maybe you should stick with Resque. If the performance difference between Shoryuken and Sidekiq matters to you, go with Sidekiq.

The point I'm trying to make is that in general for background jobs in Ruby, IMO Shoryuken/SQS is the new main option to go. It's fast, durable, distributed, auto scalable, cheap (or even free) and you don't need to care about the infrastructure, AWS will handle that for you.