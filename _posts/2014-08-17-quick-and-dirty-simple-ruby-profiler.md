---
layout: post
title: "Quick and Dirty: Simple Ruby Profiler"
---


You know when your code work well in test, development and staging environments, but when you roll out to production you notice it very slow? Even testing it pre-production, only production has the real data and load. Sad panda!

As Ruby is so awesome, there is a super easy way to find slowness in our code. I'm calling it a profiler, but nothing fancy, just a super simple hack to measure the methods execution time, no [memory, complexity etc measurements](https://en.wikipedia.org/wiki/Profiling_(computer_programming\)).

Let's do it! Go to your special production host, the one you use to test in production (testing like a boss). We know, everyone has a production host which is used for bad things.

Paste the snippet below in something available to the class scope you want to profile.

```ruby
def log_time
  beginning_time = Time.now

  result = yield

  end_time = Time.now

  info "#{caller[0]} - Completed in #{(end_time - beginning_time) * 1000} ms"

  result
end
```


Wrap the code you want to profile:

```ruby
class UserRegistration
   def register
     log_time { do_something }

     if log_time { something? }
       # ...
     elsif some_value = log_time { do_something_else }
       # ...
     end
   end
end
```

Then you will get something like that in your logs output:

> /data/app/releases/.../user_registration.rb:3:in register' - Completed in 300.00 ms
> /data/app/releases/.../user_registration.rb:5:in register' - Completed in 500.00 ms
> /data/app/releases/.../user_registration.rb:10:in register' - Completed in 2000.00 ms


And as expected, your database will be guilty in most cases. MOAR indexes.
