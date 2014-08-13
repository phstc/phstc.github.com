---
layout: post
title: Quick and Dirty: Ruby interpolation
---

Hey, it's been a while since I last wrote something here.

I would like to share something that is trivial and everybody knows, but always when I see or code in this way, it seems cool and smart.

Let's look at some `foo` code.

    def generate_awesome_key(some_object)
      # [customer].[server].[source].[msg].[dest].awesome_key

      "#{some_object.customer}.#{ENV['SERVER_NAME']}.some_object.source}.#{some_object.message_name}.#{some_object.destination_name}.awesome_key"
    end

(If you guessed that it is something to generate [StatsD](https://github.com/etsy/statsd) key/bucket path, you got it.)

That's how interpolation works: some magic double quotes `""` + `#{}` + expression.

    # hello.rb
    name = ARGV[0]
    puts "Hello World, #{name}"

    # ruby hello.rb Pablo
    # => Hello World, Pablo

Even though interpolation is the first manner that comes to mind when we need to concatenate strings + variables, in the first example the code is hard to read. I'm not a big fan of code comments in general (don't misunderstand it, please), but in this case, it makes sense due to the code readability.

There are lot of ways to write (refactor) this code - that's how Ruby works. We can do the same thing in a variety of ways, most of them really bad, but some valuable.

One of them which I demonstrate here is with arrays.

    def generate_awesome_key(some_object)
      # [customer].[server].[source].[msg].[dest].awesome_key

      [ some_object.customer,
        ENV['SERVER_NAME'],
        some_object.source,
        some_object.message_name,
        some_object.destination_name,
        'awesome_key' ].join '.'
    end


Or more "generic".

    def generate_awesome_key(some_object, *keys)
      # [customer].[server].[source].[msg].[dest].[keys…]

      ([ some_object.customer,
        ENV['SERVER_NAME'],
        some_object.source,
        some_object.message_name,
        some_object.destination_name,
       ] + keys).join '.'
    end

    # generate_awesome_key(some_object, 'awesome1', 'awesome2')
    # => customer.server.source.msg.dest.awesome1.awesome2
