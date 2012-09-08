---
layout: post
title: "Using SSH Tunneling on AWS EC2 to receive API callbacks"
category: 
tags: 
- EC2
- SSH Tunnel
- Callbacks
- English posts
---
{% include JB/setup %}

Some APIs use callbacks in their communication process, e.g. payment gateways. They usually use callbacks to notify when a payment status was changed (STARTED, ANALYSING, AUTHORIZED, COMPLETED etc).

But in the development environment our application is under "localhost", meaning the API providers couldn't reach our application.

A good and easy way to enable external callbacks to reach our internal application is using [SSH Tunneling](http://en.wikipedia.org/wiki/Tunneling_protocol).

## Enabling SSH Tunneling

The steps bellow assume that you are familiar with [AWS EC2](http://aws.amazon.com/ec2/) and have an EC2 Linux based instance running.

###Security Groups

In AWS Web Console go to Security Group and enable port 8100. This port will be used by callbacks.

In your API Provider Administration or in the API Request message (it depends on the API Provider) - configure ```ec2-100-10-100-10.compute-1.amazonaws.com:8100/AWESOME-CALLBACK-PATH``` as your callback url.

```ec2-100-10-100-10.compute-1.amazonaws.com``` is your EC2 instance Public DNS.

###In your EC2 Instance

Open ```/etc/ssh/sshd_config``` uncomment the line GatewayPorts or create one if it doesn't exist and set it to "yes"

    # /etc/ssh/sshd_config
    GatewayPorts yes

###In your machine "localhost"

Execute

    $ ssh -N -R 8100:localhost:3000 -i YOUR_AWS.pem YOUR_USER@ec2-100-10-100-10.compute-1.amazonaws.com

The port 3000 is the local port of your application.

Don't forget to change YOUR_AWS.pem and YOUR_USER to your specific parameters.

If everything worked as expected, when you open http://ec2-100-10-100-10.compute-1.amazonaws.com:8100, your local application will be invoked.

