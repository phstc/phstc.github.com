---
layout: post
title: "Opening a Rails console with Capistrano 3"
---


## Opening a Rails console

To open a Rails console with Capistrano add the snippet below in your `config/deploy.rb`.

```ruby
namespace :rails do
  desc 'Open a rails console `cap [staging] rails:console [server_index default: 0]`'
  task :console do
    on roles(:app) do |server|
      server_index = ARGV[2].to_i

      return if server != roles(:app)[server_index]

      puts "Opening a console on: #{host}...."

      cmd = "ssh #{server.user}@#{host} -t 'cd #{fetch(:deploy_to)}/current && RAILS_ENV=#{fetch(:rails_env)} bundle exec rails console'"

      puts cmd

      exec cmd
    end
  end
end
```

Usage:

```bash
# To open the first server in the servers list
cap [staging] rails:console

# To open the second server in the servers list
cap [staging] rails:console 1
```

## BONUS 1: Opening a SSH session

```ruby
desc 'Open ssh `cap [staging] ssh [server_index default: 0]`'
task :ssh do
 on roles(:app) do |server|
   server_index = ARGV[2].to_i

   return if server != roles(:app)[server_index]

   puts "Opening a console on: #{host}...."

   cmd = "ssh #{server.user}@#{host}"

   puts cmd

   exec cmd
 end
end
```

Usage:

```bash
# To open the first server in the servers list
cap [staging] ssh

# To open the second server in the servers list
cap [staging] ssh 1
```

## BONUS 2: Opening multiple SSH sessions

Make sure you have [csshx](https://code.google.com/p/csshx/) installed. On OS X: `brew install csshx`.

```ruby
desc 'Open csshx `cap [staging] csshx`'
task :csshx do
  servers = []
  on roles(:app) do |server|
    servers << "#{server.user}@#{host}"
  end

  cmd = "csshx #{servers.join(' ')}"

  puts cmd

  exec cmd
end
```

Usage:

```bash
cap [staging] csshx
```

