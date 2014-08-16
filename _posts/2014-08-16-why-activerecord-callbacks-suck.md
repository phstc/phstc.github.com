---
layout: post
title: "Why ActiveRecord callbacks suck?"
---

It's a kind of cliché in the Rails community to associate ActiveRecord callbacks with something bad or wrong.

Given that every time I start typing `after_` or `before_`, I hear a voice in my head: "Are you sure?". This voice usually attempts me to fight the evil (poor design choices) with more evil (++poor design choices).

Before attempting an improvement to remove or avoiding to write a callback, I need to pause and remember why or when they suck.

IMHO they can suck, because it's easy to use ("syntactic sugar")  them to violate the [Single Responsibility Principle (SRP)](https://en.wikipedia.org/wiki/Single_responsibility_principle) and they can probably slow down your tests.

## Violating the Single Responsibility Principle (SRP)

If you follow SRP by the book, you might end up with hundreds of single-public-method classes, and it isn't what we want.

By SRP I'm assuming: If you need to send a confirmation email every time you create a new user (such a trivial example), you have two responsibilities: one to create a user and another to send an email and they need specific classes and methods. If you do it all in the same place class/method (violating SRP), it can result in unexpected processing depending on your context or hard to maintain, because to update one code you need to change the other consequently.

Considering my assumption above, the code below looks good:

    class User < ActiveRecord:Model
      after_create :send_confirmation_email
    
      def send_confirmation_email
        UserMailer.registration_confirmation(self).deliver
      end
    end

We have two classes: `User` and `UserMailer` and two methods: `User#save` and `UserMailer.registration_confirmation` to create a user and send an email respectively. 

When you have a website allowing user registrations, you expect that something (an email) can happen when a user is created. Although I said it's "looking good", you will have a problem if you want to create a new user in the console without sending the confirmation email, you can't. The user creation is coupled with the email sending, so it isn't the best design choice, but how frequently do you need to create a user without sending a confirmation email? Does it deserve a `UserRegistration`?

    class UserRegistration
      def initialize(user)
        @user = user
      end
    
      def register
        @user.save
        UserMailer.registration_confirmation(self).deliver
      end
    end

Being honest, I would go with a `UserRegistration` class as an improvement on demand, based on business logic needs (i.e. a requirement to allow users creation without sending the confirmation email). If you do it prematurely, someone can use `User#save` instead of `UserRegistration.new(user).register`, just because it is what he does "by default" using Rails, so that innocently skipping the confirmation email by mistake.

I believe the problem above is one of the simplest problem with callbacks, but frequently on our daily basis. So before over-engineering it, ignore the voice in your head, check your context, ask why and when. 

Violating SRP hurts when it makes things unexpected and hard to understand (hard to debug).

Imagine every time you create an order, besides persisting it you need also to synchronise it with an ERP (API/HTTP call), send a message to a queue (to send an email) and update products. And now, imagine if all this stuff (or even more) is done by an `after_save`, called after an unpretentious `Order#save`. Would you expect that?

A new developer joins the project, starts a task to update all orders `origin` attribute based on other order attributes.

    Order.all.find_each do |order|
      # update `origin`
      order.save
    end

> Fuck this. Fuck that. Fuck those too. Fuck all these. This thing in particular.

That's what is going to happen. That's is one of the reasons that callbacks can suck, having more responsibilities than the expected.

What's expected and unexpected is very relative, I know. But I suggest to consider much more your context, than just framework based classes or method names.


# Slow down your tests

Callbacks can slow down your tests (RSpec etc), not your production code. If you need to do more stuff after persisting an order, no matter how you do it:

    class Order < ActiveRecord::Model
      protect :save
      
      def place
        # do something else
        save
      end
    end

Creating new methods and changing others visibility, creating other classes/methods, observers, **callbacks** etc. 

You need to do that. So when a user click on "Place Order", he will need to wait all this processing, no matter which abstractions were used to implement it. That's it.

Why it can slow down your tests? Imagine you create orders in many many tests, and the `Order#save` do something magical using an `after_save` then you forgot to stub it `before` running your tests. Your orders creation will be taking much longer to finish, because it's doing something you don't need in most of your tests. If you use like a `UserRegistration` you will have the code and specs separately for each responsibility, win! Win? Would you do this improvement (it's an improvement, better code for sure!) to make your tests easier? Would you change your code for your tests or to meet business logic needs?

Unrelated to slowness, "luckily" your tests should fail when there are lot of callbacks behind the scenes doing weird things (helping to spot the stub candidates), because they probably need credentials that are only available in other environments. If the credentials are also available in test environment you will probably run into other unexpected issues.

## Sum up

Callbacks "can" suck, correct! But avoiding them without considering the context, just because they are "bad" can suck more.