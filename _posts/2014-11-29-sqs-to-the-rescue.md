---
layout: post
title: SQS to the rescue
---

**tl;dr** [Amazon SQS](https://aws.amazon.com/sqs/) is currently one of the best options for a queuing service. It's [inexpensive
](https://aws.amazon.com/sqs/pricing/) (first 1 million requests per month are free, thereafter $0.50 per 1 million requests), fast, reliable, scalable, fully managed message queuing service.  It just works.


For the follow examples I'm using the [aws-sdk-ruby](https://github.com/aws/aws-sdk-ruby), which is a great SDK, because it's transparent to the [official API documentation](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/Welcome.html), no abstractions, no different method names etc, what you see on the documentation is what you get on the sdk.

## Getting your hands dirty

Let's begin by bootstrapping a queue using the [create](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/SQS/QueueCollection.html#create-instance_method) method:

```ruby
sqs.queues.create 'myqueue', visibility_timeout: 600, message_retention_period: 345600
```

Then let's send a message using the [send_message](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/SQS/Queue.html#send_message-instance_method) method:

```ruby
sqs.queues.named('myqueue').send_message 'HELLO'
```

Finally let's consume a message using the [receive_message](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/SQS/Queue.html#receive_message-instance_method) method:

```ruby
sqs_msg = sqs.queues.named('myqueue').receive_message
do_something(sqs_msg)
sqs_msg.delete
```

Catching things up - we created a queue that holds messages for 4 days `message_retention_period: 345600`, which means: if we don't consume a message within 4 days the message will be lost.

The `visibility_timeout` is the time in seconds we have to consume a message after its reception. When we send a message to SQS `send_message`, the message is listed on "Messages Available", when we receive a message `receive_message` the message is moved to "Messages in Flight", if we don't `delete` the message within 600 seconds (which is the visibility timeout we configured above) the message will be moved back to "Messages Available" and become ready to be consumed again. [Read more about the SQS Message Lifecycle](https://aws.amazon.com/sqs/details/#Amazon_SQS_Message_Lifecycle).

Be generous while configuring the default `visibility_timeout` for a queue. If your worker in the worst case takes 2 minutes to consume a message, set the `visibility_timeout` to at least 4 minutes. It doesn't hurt and it will be better than having the same message being consumed more than the expected.

## Auto retrying errors

```ruby
begin
  sqs_msg = sqs.queues.named('myqueue').receive_message
  do_something(sqs_msg)
  sqs_msg.delete
rescue => e
  error e
end
```

In the code above if `do_something` raises an exception, the message will become available again as soon as its `visibility_timeout` expires, so other workers will be able to re-attempt the message.

This behaviour will also cover in case your process or server crashes. Basically if you don't call `sqs_msg.delete`, the message will become available again no matter what :)

## Dead Letter Queues

Let's retry stuff, but not forever, right? Sometimes no matter how many times we re-attempt a message, it will continue failing. To do not lose or retry forever these messages addicted to failure, we can let them rest in peace using a [Dead Letter Queue
](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/SQSDeadLetterQueue.html).

The code below creates `myqueue_failures` and associates it with `myqueue` as a dead letter queue:

```ruby
dl_queue = sqs.queues.create('myqueue_failures')

sqs.queues.create('myqueue', redrive_policy: %Q{ {"maxReceiveCount":"3", "deadLetterTargetArn":"#{dl_queue.arn}"}" })
```

The `redrive_policy` above tells SQS to move a message from `myqueue` to `myqueue_failures` if its receive count reaches 3.


## Delaying a message `perform_in`

SQS supports delaying a message up to 15 minutes, before it becomes available to be consumed using the  [delay_seconds](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/SQS/Queue.html#send_message-instance_method) option:

```ruby
sqs.queues.named('myqueue').send_message 'HELLO', delay_seconds: 60
```

But as we are all [hackerz](http://www.imdb.com/title/tt0113243/) and we know that the `visibility_timeout` [supports up to 12 hours](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ChangeMessageVisibility.html), we can use it to extend the delay.

### Sending a message with extended delay

```ruby
perform_at = Time.now + 1.hour # max: 12.hours

sqs.queues.named('myqueue').send_message 'H311O', 
  message_attributes: { 
    'perform_at' => { string_value: perform_at.to_s, 
                      data_type: 'String' }
```

### Receiving a message with extended delay

```ruby
sqs_msg = sqs.queues.named('myqueue').receive_message, message_attribute_names: ['perform_at']

delay = Time.parse(sqs_msg.message_attributes['perform_at'][:string_value]) - Time.now

if delay > 0
  sqs_msg.visibility_timeout = delay.to_i
else
  do_something(sqs_msg)
  sqs_msg.delete
end
```

Be careful with this workaround, because it will increase the message receive count.

## 1 million != 1 million

> [first 1 million requests per month are free, thereafter $0.50 per 1 million requests](https://aws.amazon.com/sqs/pricing/))

1 million requests doesn't mean 1 million jobs consumed, as we need at least 3 requests to fully consume a message:

1. `send_message`
2. `receive_message`
3. `sqs_msg.delete`

Although these requests can be executed in batches up to 10 messages, you will need all of them to consume a message, and even if you are using the [poll](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/SQS/Queue.html#poll-instance_method) method with a high `wait_time_seconds`, you will probably make some empty requests while looking for new messages. 

## SNS to SQS

Sending the same message to more than one queue using SQS:

```ruby
msg = order.json

# to send an email to the customer
sqs.queues.named('order_email_queue').send_message msg
# to integrate with an ERP
sqs.queues.named('order_erp_queue').send_message msg
```

Using [SNS to SQS](http://docs.aws.amazon.com/sns/latest/dg/SendMessageToSQS.html):

```ruby
msg = order.json

topic.publish(msg)
# sends to order_email_queue and order_erp_queue
```

[SNS will fanout the message to both queues](http://docs.aws.amazon.com/sns/latest/dg/SNS_Scenarios.html) - consequently you will [pay](https://aws.amazon.com/sns/pricing/) (SNS to SQS is free) and wait for only one request to SNS, instead of two to SQS.

## Trouble in paradise

SQS is really good, the Ruby SDK is great, but… how to consume messages continuously?

```ruby
# myqueue_worker.rb
sqs.queues.named('myqueue').poll { | sqs_msg | do_something(sqs_msg) }
```

Then:

```bash
ruby myqueue_worker.rb
```

Doesn't seem to be very reliable, right? No threads, consumes messages one by one, and if you need to consume from other queue you will need to start a new process `ruby myotherqueue_worker.rb`, which is waste of resources when one queue is empty and others have messages available.

### Introducing Shoryuken

[Shoryuken](https://github.com/phstc/shoryuken) is a project based on [Sidekiq](https://github.com/mperham/sidekiq), that uses all the cool stuff Sidekiq did with processes, [Celluloid](https://github.com/celluloid/celluloid), cli etc, but for SQS.

No more `ruby myotherqueue_worker.rb`, [Shoryuken will handle that for you](https://github.com/phstc/shoryuken#start-shoryuken), including [signal handling](https://github.com/phstc/shoryuken/wiki/Signals).

A single Shoryuken process can consume messages from multiple queues, [load balancing the consumption](https://github.com/phstc/shoryuken#load-balancing). 

It's transparent to SQS, it passes the [ReceivedMessage](docs.aws.amazon.com/AWSRubySDK/latest/AWS/SQS/ReceivedMessage.html) to the [workers](https://github.com/phstc/shoryuken#worker-class), no abstractions, no Rescue compatibility, SQS as is!

It accepts [multiple workers per queue](https://github.com/phstc/shoryuken/wiki/Sending-a-message#multiple-workers-for-the-same-queue).

Check more on the [project's README](https://github.com/phstc/shoryuken).

## Conclusion

SQS is cheap, you don't need to worry about managing your queue services, setup Redis etc. 

Keep your focus on your workers/jobs (which should be the most important to you) and let Amazon to take care of the infrastructure, it works!