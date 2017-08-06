---
layout: post
title: Using Ruby and Amazon SQS FIFO Queues
---

For those not familiar with [Amazon SQS](https://aws.amazon.com/sqs/), it's a reliable and high-scalable fully managed queue service. If you need to process in background without the overhead of managing your queue service, and all that jazz, go for it. It can be also [free depending on our usage](https://aws.amazon.com/sqs/pricing/).

This post is about [Amazon SQS FIFO Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html), which is a type of queue designed to guarantee that messages are processed exactly once and in the exact order that they are sent.

## Shoryuken

The examples in this post are based on [Shoryuken](https://github.com/phstc/shoryuken), a SQS thread based message processor in Ruby.

Why Shoryuken? [aws-sdk-ruby](https://github.com/aws/aws-sdk-ruby) is great, but it's just an API client. Shoryuken empowers the SDK by adding support for continuously polling and thread based processing, besides that, it also adds abstractions facilitating the integration with SQS, including [Active Job support](https://github.com/phstc/shoryuken/wiki/Rails-Integration-Active-Job). 

## FIFO Queues

There are two key attributes when working with FIFO Queues: Message Group ID and Message Deduplication ID.

### Message Group ID

Message Group ID is required when sending messages to a FIFO Queue, you can either use the same message group for all messages, or group them by some business logic, let's say `account_id`, or whatever makes sense to you. The messages will be received in the order they were sent by group.

Shoryuken automatically sets the Message Group ID to `ShoryukenMessage` if it is not provided.

```ruby
# Shoryuken will set the message_group_id to ShoryukenMessage
MyWorker.perform_async('test') 

# User-defined message_group_id
MyWorker.perform_async('test', message_group_id: '...')
```

See [Message Group ID](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html#FIFO-queues-message-order).

### Message Deduplication ID

For exactly-once processing, FIFO Queues use the Message Deduplication ID attribute for identifying duplicates within a 5 minutes interval. The Message Deduplication is auto generated based on the message body if you enable content-based deduplication when creating your queue, if not enabled, it must be set when sending messages.

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
shoryuken sqs create myqueue.fifo
```

#### Create a worker

```ruby
class MyWorker
  include Shoryuken::Worker

  shoryuken_options queue: 'myqueue.fifo', auto_delete: true

  def perform(sqs_msg, body)
    puts body
  end
end
```

#### Send messages

```ruby
MyWorker.perform_async('test1', message_group_id: 'group1')
MyWorker.perform_async('test2', message_group_id: 'group1')
MyWorker.perform_async('test3', message_group_id: 'group1')
```

#### Start Shoryuken


```shell
shoryuken -q myqueue -r ./myworker.rb
```

### Process messages one by one

By default, Shoryuken tries to receive as many messages as it can, limited to the number of the available threads (see [concurrency](https://github.com/phstc/shoryuken/wiki/Shoryuken-options#concurrency)). If there are 10 available threads, it may receive up to 10 messages for the same Message Group ID and process them all in parallel, not one by one. 

If you want to enforce one by one, set `max_number_of_messages: 1` (see [receive options](https://github.com/phstc/shoryuken/wiki/Receive-Message-options)) then if you send messages as follows:

```ruby
MyWorker.perform_async('A', message_group_id: 'group1')
MyWorker.perform_async('B', message_group_id: 'group1')
MyWorker.perform_async('C', message_group_id: 'group1')
```

Shoryuken will receive `A`, process, receive `B`, process, receive `C` and process, one by one.

### How about multiple processes and other clients? 

Every time Shoryuken (or any other SQS client) receives a message associated with a Message Group ID, SQS locks the group until the message gets deleted or its visibility timeout expires. Preventing other Shoryuken processes or SQS clients to receive messages for the group.

## Should I use FIFO Queues?

#### If you don't want duplicated messages within a 5 minutes interval.

Yes.

#### If you want to control the order that the messages are received.

Yes.

#### If you want to control the order, but you want to keep processing duplicates.

Yes. Just set `message_deduplication_id` to something unique (`SecureRandom.uuid`) and SQS won't consider it as a duplicate.

#### If you don't care about the order and duplicates.

No. Go with standard queues.