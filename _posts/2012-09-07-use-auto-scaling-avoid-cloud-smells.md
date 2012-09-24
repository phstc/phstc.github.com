---
layout: post
title: "Use Auto Scaling, avoid Cloud Smells"
category: 
tags: [AWS, Auto Scaling, Cloud Smells]
---
{% include JB/setup %}

A Cloud hosting has two main characteristics:

* Pay-as-you-go

* Scale Resources up or down based on demand (Elastic Architecture)

These are the features that differentiate it from a traditional hosting.

If you're in the Cloud, you should also follow these characteristics.

To be in the Cloud:

* You must pay only for the resources needed for your application without waste

* Your application must be able to scale up and down on demand

You can host an application in a Cloud, but it doesn't mean that your application is Cloud capable (Cloudable). In the same way that your [Code can Smell](http://en.wikipedia.org/wiki/Code_smell), your Cloud can Smell too.

## Pay as you go

Cloud hosting typically charges by minute, hour, storage (MB, GB etc) or bandwidth. The bandwidth is usually the villain. You might think *"The instance is cheap, the storage is cheap, it is really cool. I'm totally in love with it"*, but when your application starts to grow, you will probably notice that your CDN ([CloudFront](http://aws.amazon.com/cloudfront)) is not as cheap as it was before etc.

Traditional hosting typically charges by longer periods (monthly, semiannually, annually etc).

## Scale Resources up or down based on demand (Elastic Architecture)

Elastic Architecture allows you to add or remove resources on demand. If your website needs more resources during the day you can scale up, but if at night it doesn't need these resources you can easily scale down. 

Basically, if you have more than one instance, you should use Auto Scaling. Normal websites don't have the same traffic at night as during the day. If all of your instances are running 24x7, it probably means that your Cloud Smells.

Traditional hosting is more straight. They usually require contracts to provide resources. You can't easily add or remove them.

## Auto Scaling

Amazon AWS offers [Auto Scaling](http://aws.amazon.com/autoscaling), which allows you to automatically scale up or down your application according your needs.

Steps to use AWS Auto Scaling:

1. *Create a launch configuration*. The launch configuration are parameters to launch instances, such as image (AMI), instance type, security group, key pair and user data.

2. *Create an auto scaling group*. The auto scaling group defines the auto scaling conditions, minimum and maximum number of instances, available zones, default cool down and load balancer (ELB).

3. *Create a scaling policy*. The scaling policy defines how the auto scaling should scale up or down, if it is by size or percent and the cool down.

4. *Create an alarm metric*. The alarm metric is the metric used to decide when to increase or decrease the instances. Usually by response time. The metrics are provided by [CloudWatch](http://aws.amazon.com/cloudwatch).

5. *Create an alarm request*. It is the last step to setup the Auto Scaling. The alarm request defines how to measure the metrics defined by the alarm metric, for example the time and period to evaluate the metric.

You can setup Auto Scaling using [Command Line Tools, Java, Ruby etc](http://aws.amazon.com/developertools).

### Auto deployable applications

If your application never changes or doesn't change frequently, you can create an image with your application, but in general it isn't true, applications usually change frequently.

Although it isn't obligatory to be auto deployable, I strongly recommend to turn your application able to be deployed by scripts without manual intervention.

For auto deployable applications in the Auto Scaling launch configuration you can define an user data (it is a script) to get a fresh version of your application. If your application is easy to deploy, maybe just a git clone is enough, if your application has several steps to be deployed, you can create a bash script or download a "RPM" etc.

You can also setup your infrastructure in the user data by using [Puppet](http://puppetlabs.com), [Chef](http://www.opscode.com/chef) or [Amazon Cloud Formation](http://aws.amazon.com/cloudformation).

You can also mix the infrastructure and deployment automation, but it usually takes longer to create new instances.

## Some advices

### Don't name your instances

> You should change your instances as you change your underwear. [Lucas Teixeira](https://twitter.com/lucastex)

I know that is cool to name your instances with hipster names, but avoid it. It only leads you to create non-automated systems and applications. Don't be attached to your instances, it is a Cloud Smell.

For each deploy of your application, you should create a new and  fresh instance.

### Logging

An important advice is to use [Rsyslog](http://en.wikipedia.org/wiki/Rsyslog) or other similar tool. If you have more than one instance running, you will need a centralized logging system. 

### Scripting

You can write bash scripts to manage your instances, auto scaling etc using [Command Line tools](http://aws.amazon.com/developertools). They are really easy to use, but I recommend to create them with languages that you are strongly familiar with it. In the beginning bash scripts seem to be easy, but as they grow, and they will, they start to be hard to maintain.

I developed with Lucas Teixeira ["scripts"](https://github.com/lucastex/aws-tools) in Java to create instances and auto scaling automation. They are a plain implementation, not an abstraction. I recommend them to learn "Auto Scaling guided by Code". Why in Java? I don't have any idea, maybe nostalgic feelings. Lucas started it in Java, then I followed him.

