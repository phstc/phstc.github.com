---
layout: post
title: Using Ruby and Amazon SQS FIFO Queues
---

For those not familiar with [Amazon SQS](https://aws.amazon.com/sqs/), it's a reliable and high-scalable fully managed queue service. If you need to process in background, and all that jazz, but without the overhead of managing your queue service, go for it. It can be also [free depending on our usage](https://aws.amazon.com/sqs/pricing/).

This post is about [Amazon SQS FIFO Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html), which is a type of queue designed to guarantee that messages are processed exactly once and in the exact order that they are sent.

## Shoryuken

The examples in this post are based on [Shoryuken](https://github.com/phstc/shoryuken), a SQS thread based message processor in Ruby.

Why Shoryuken? [aws-sdk-ruby](https://github.com/aws/aws-sdk-ruby) is great, but it's just an API client. Shoryuken empowers the SDK by adding support for continuously polling and thread based processing, besides that, it also adds abstractions simplifying the integration with SQS, including [Active Job support](https://github.com/phstc/shoryuken/wiki/Rails-Integration-Active-Job). 

## FIFO Queues

There are two key attributes when working with FIFO Queues: Message Group ID and Message Deduplication ID.

### Message Group ID

For the Message Group ID you can either use the same message group for all messages, or group them by some business logic, let's say `account_id`, or whatever that makes sense to you. The messages will be received in the order they were sent to their group.

Shoryuken automatically sets the Message Group ID to `ShoryukenMessage` if it is not provided.

```ruby
# Shoryuken will set the message_group_id to ShoryukenMessage
MyWorker.perform_async('test') 

# User-defined message_group_id
MyWorker.perform_async('test', message_group_id: '...')
```

See [Message Group ID](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html#FIFO-queues-message-order).

### Message Deduplication ID

For exactly-once processing, FIFO Queues use the Message Deduplication ID for identifying and rejecting duplicates sent within a 5 minutes interval. The Message Deduplication is auto generated based on the message body if you enable content-based deduplication when creating your queue.

Shoryuken automatically sets the Deduplication ID to a SHA-256 hash of the message body if it is not provided.

```ruby
# Shoryuken will set the messaged_deduplication_id to a SHA-256 hash using the message body
MyWorker.perform_async('test') 

# User-defined message_deduplication_id
MyWorker.perform_async('test', message_deduplication_id: '...')
```

See [Message Deduplication ID](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html#FIFO-queues-exactly-once-processing).


### Getting started

A few steps for using message groups and Shoryuken.

#### Create a FIFO Queue

```shell
shoryuken sqs create queue.fifo
```

#### Create a worker

```ruby
class HelloWorker
  include Shoryuken::Worker

  shoryuken_options queue: 'queue.fifo', auto_delete: true

  def perform(sqs_msg, name)
    puts "Hello, #{name}"
  end
end
```

#### Send messages

```ruby
HelloWorker.perform_async('Ken 1', message_group_id: 'hello')
HelloWorker.perform_async('Ken 2', message_group_id: 'hello')
HelloWorker.perform_async('Ken 3', message_group_id: 'hello')
```

#### Start Shoryuken

```shell
shoryuken -q queue -r ./hello_worker.rb
```

That's it. Shoryuken will start processing your messages.


### Sequential processing per group

By default, Shoryuken tries to receive as many messages as it can, limited to the number of the available threads (see [concurrency](https://github.com/phstc/shoryuken/wiki/Shoryuken-options#concurrency)) and the SQS limit of 10 messages per request. If there are 10 available threads, it may receive up to 10 messages for the same Message Group ID and process them all in parallel. 

If you want to sequentially process one message at a time, set `max_number_of_messages: 1` (see [receive options](https://github.com/phstc/shoryuken/wiki/Receive-Message-options)), with that, if you send messages as follows:

```ruby
ExpireMembershipsWorker.perform_async(date)
ActivatePendingMembershipsWorker.perform_async(date)
RenewMembershipsWorker.perform_async(date)
```

Shoryuken will receive the message for `ExpireMembershipsWorker`, process, receive the message for`ActivatePendingMembershipsWorker`, process, receive the message for `RenewMembershipsWorker` and process, one by one.

### How about multiple processes and other clients? 

Every time Shoryuken (or any other SQS client) receives a message associated with a Message Group ID, SQS locks the group until the message gets deleted or its visibility timeout expires. Preventing Shoryuken or other SQS clients to receive messages for the group.

### Using FIFO Queues for preventing rate limits

Supposing you are doing something in background that calls an external API that limits the number of requests per second. You could use FIFO for that.

```ruby
Shoryuken.sqs_client_receive_message_opts = {
  max_number_of_messages: 1
}

class SyncWithExternalAPIWorker
  include Shoryuken::Worker

  shoryuken_options queue: 'queue.fifo', auto_delete: true

  def perform(sqs_msg, name)
    # do something...

    # sleep for 100ms, limiting the max number of calls per second to 10
    sleep(0.1)
  end
end
```

I don't recommend long running workers, I would go with `sleep` only for short delays. If you need longer delays, you can try to take advantage of the [visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html) or  [send messages with a delay](https://github.com/phstc/shoryuken/wiki/Sending-a-message#delaying-a-message).

### Should I use FIFO Queues?

##### If you want to reject duplicated messages sent within a 5 minutes interval.

Yes.

##### If you want to control the order that the messages are received.

Yes.

##### If you want to control the order, but you want to receive duplicates.

Yes. Just set `message_deduplication_id` to something unique (`SecureRandom.uuid`) and SQS won't reject as a duplicate.

##### If you don't care about the order and duplicates.

No. Go with standard queues.