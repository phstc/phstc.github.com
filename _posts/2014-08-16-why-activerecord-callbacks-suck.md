---
layout: post
title: "Why ActiveRecord callbacks suck?"
---

It's a kind of cliché in the Rails community to associate ActiveRecord callbacks with something bad or wrong.

Given that every time I start typing `after_` or `before_`, I hear a voice in my head: "Are you sure?". This voice usually attempts me to fight the evil (poor design choices) with more evil.

Callbacks are a sugar syntax to violate the [Single Responsibility Principle (SRP)](https://en.wikipedia.org/wiki/Single_responsibility_principle) and they can slow down your tests. That's why I believe they can suck, but it doesn't happen all the time. So before attempting an improvement to remove or avoiding to write a callback, pause a think if the responsibility you want to add belongs to the save context.

## Violating the Single Responsibility Principle (SRP)

If you follow SRP by the book, you might end up with hundreds of single-public-method classes, and it isn't what we want.

If you need to send a confirmation email every time you create a new user, you have two responsibilities: one to create a user and another to send an email and they need specific classes and methods. If you do it all in the same place class/method, it can result in unexpected processing **depending on your context** or become hard to maintain, because to update one code you need to change the other consequently.

```ruby
class User < ActiveRecord:Model
  after_create :send_confirmation_email

  def send_confirmation_email
    UserMailer.registration_confirmation(self).deliver
  end
end
```

We have two classes: `User` and `UserMailer` and two methods: `User#save` and `UserMailer.registration_confirmation` to create a user and send an email respectively.

With this design if you want to create a new user in the console without sending the confirmation email, you can't. The user creation is coupled with the email sending, so it isn't the best design choice, but how frequently do you need to create a user without sending a confirmation email? Does it deserve a `UserRegistration`?

```ruby
class UserRegistration
  def initialize(user)
    @user = user
  end

  def register
    @user.save
    UserMailer.registration_confirmation(self).deliver
  end
end
```

Being honest, I would go with a `UserRegistration` class as an improvement on demand, based on business requirements (i.e. a requirement to allow users creation without sending the confirmation email). If you do it prematurely, someone can use `User#save` instead of `UserRegistration.new(user).register`, just because it is what he does "by default" using Rails, so that innocently skipping the confirmation email. I prefer to keep the common usage simple and trivial as possible and only treat the exceptions with special code.

I believe the problem above is one of the simplest problem with callbacks, but frequently on our daily basis. So before over-engineering it, ignore the voice in your head and think about it.

Violating SRP hurts when it makes things unexpected, hard to understand (hard to debug).

Imagine every time you create an order, besides persisting it you need also to synchronise it with an ERP (API/HTTP call), send a message to a queue (to send an email) and update products. And now, imagine if all this stuff (or even more) is done by an `after_save`, called after an unpretentious `Order#save`. Would you expect that?

A new developer joins the project, starts a task to update all orders `origin` attribute based on other order attributes.

```ruby
Order.all.find_each do |order|
  # update `origin`
  order.save
end
```

> Fuck this. Fuck that. Fuck those too. Fuck all these. This thing in particular.

That's what is going to happen. That's is one of the reasons that callbacks can suck, having more responsibilities than the expected.

What's expected and unexpected is very relative, I know. But I suggest to consider much more your context, than just framework based classes or method names.


# Slow down your tests

Callbacks can slow down your tests (RSpec etc), not your production code. If you need to do more stuff after persisting an order, no matter how you do it:

```ruby
class Order < ActiveRecord::Model
  protect :save

  def place
    # do something else
    save
  end
end
```

Creating new methods and changing others visibility, creating other classes/methods, observers, **callbacks** etc.

You need to do that. So when a user click on "Place Order", he will need to wait all this processing, no matter which abstractions were used to implement it. That's it.

Why it can slow down your tests? Imagine you create orders in many many tests, and the `Order#save` does something magical using an `after_save` then you forgot to stub it `before` running your tests. Your orders creation will take much longer to finish, because they are doing something you don't need in most of your tests. If you use like a `UserRegistration` you will have the code and specs separately for each responsibility, win! Win? Would you change your production code just to make your tests faster?

If callbacks start to slow down your production code, I would recommend to return to the **Violating the SRP** topic, probably you are using them in the wrong way.

## Sum up

Callbacks "can" suck, correct! But avoiding them without considering the context, just because they are "bad" can suck more.