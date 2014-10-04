---
layout: post
title: "Auto scale Sidekiq workers on Amazon EC2"
---

We use [Sidekiq](https://github.com/mperham/sidekiq) a lot, from image conversion to shipment tracking numbers generation. 

The size of our queues change drastically during the day. When it happens, we increase or decrease our workers by adding or removing EC2 instances automatically, thanks to [Amazon Auto Scaling](http://aws.amazon.com/autoscaling/).

## Amazon Auto Scaling

I will not get into the details of Amazon Auto Scaling, as it deserves an entire post about it.

Roughly you have to create an Auto Scaling Group, Launch Configuration and Scaling Up/Down policies.

Remember that Amazon Auto Scaling will not remove your instances automatically. If you create Scaling Up policies, you will need to create Scaling Down as well to remove them.

### Netflix Asgard

Instead of creating your own Auto Scale scripts with [AWS-SDK](http://aws.amazon.com/sdkforruby/), I recommend [Netflix Asgard](https://github.com/Netflix/asgard). It isn't a killer tool, but it works.

#### Asgard considerations

It doesn't work well with other Tomcat apps. To work properly with Tomcat, it must be the ROOT app. Another option is to run it as a [standalone app](https://github.com/Netflix/asgard/wiki/Quick-Start-Guide), this is how I use it.

    JAVA_HOME=/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home java -Xmx1024M -XX:MaxPermSize=128m -jar asgard-standalone.jar "" localhost 8888

(it works better on Java 6)

## Queue Size Metric

To trigger your Scale Up/Down policies based on the Queue Size, you have to publish a new [Custom Metric on CloudWatch](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/publishingMetrics.html).

### Get the Queue Size

    require "sidekiq"
    class SidekiqMetric
      def queue_size
        stats = Sidekiq::Stats.new
        stats.queues.values.inject 0, :+
      end
    end

### Publish the Queue Size

    require "aws-sdk"
    class QueueSizeMetric
      def initialize
        AWS.config access_key_id: "…", secret_access_key: "…"
      end

      def put
        metric = AWS::CloudWatch::Metric.new "Worker", "QueueSize"
        size   = SidekiqMetric.new.queue_size
        metric.put_data [{value: size}]
      end
    end



### Keep the metrics updated

We use the [whenever gem](https://github.com/javan/whenever) to create a cron job to update the queue size every minute.

    # schedule.rb
    every 1.minutes do
      command "cd /home/ubuntu/queues/current && bundle exec rake update_queue_size_metric"
    end

    # RakeFile
    task :update_queue_size_metric do
      QueueSizeMetric.new.put
    end

There are many other ways to keep the metrics updated, for example using a worker to update the metrics.

    # app/workers/queue_size_metric_worker.rb
    class QueueSizeMetricWorker
      include Sidekiq::Worker

      def perform
        QueueSizeMetric.new.put
        QueueSizeMetricWorker.perform_in 1.minute
      end
    end

The important thing is to keep them updated, no matter how. Otherwise your Auto Scaling will not behave as expected.

## All Done

Now when creating your Scale Policies, set the metric namespace to **Worker** and name to **QueueSize**.