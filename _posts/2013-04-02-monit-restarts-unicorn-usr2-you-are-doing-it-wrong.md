---
layout: post
title: "Monit restarts Unicorn USR2 - You are doing it wrong"
---

**TL;DR** Use Monit to `start` and `stop` Unicorn, if you want to graceful `restart`, do it outside Monit ([sample scripts](https://gist.github.com/phstc/5312520)).

## Monit

[Monit](http://mmonit.com/monit/) is an awesome open source tool for managing and monitoring processes, programs, files etc.

Although Monit is awesome, unfortunately for process monitoring, it hasn't a `restart` action, but only `start` and `stop` actions.

In fact, as you can see in the [Monit source code](http://mmonit.com/monit/download/), the `restart` explicit executes `stop` and `start`.

```
restart)
	$0 stop
	$0 start
```

### Monit life cycle

Monit checks for a pid or process matching (depending on your configuration) in a specific interval defined in `monitrc` (`set daemon n`) and, if the process doesn't exist, Monit will execute the `start` action.

Monit only restarts (`stop` and `start`) a process, if you manually execute `monit restart all|name` or if you defined a resource testing triggering a restart.

```
check process unicorn_master with pidfile "..."
      start = "..."
      stop = "..."
      if cpu is greater than 50% for 5 cycles then restart
```

## Unicorn

[Unicorn](http://unicorn.bogomips.org/) is a [great ass](http://www.youtube.com/watch?v=zc16ABAWTRk&feature=youtu.be&t=6m12s) Rack HTTP server.

The combination of Monit and Unicorn is the motivation for this post.

I saw several posts over the internet suggesting this configuration.

```
check process unicorn_master with pidfile "…"
  start program = "/bin/true"
  stop program = "set ruby;set gems path; kill -USR2 `cat pidfile`"
```

People use this suggested configuration above to keep Unicorn always running and restarts it gracefully.

I'm not saying that it is wrong. If it works for you, it's ok. But even if it works, don't use a long command in the Monit actions because it is hard to maintain so instead, create a bash script and set it in the action.

Unicorn accepts [signals](http://unicorn.bogomips.org/SIGNALS.html) in kill action. The signal `USR2`, reexecutes the running binary. When the new process is up and running, it kills the original one, keeping the server always running. The advantage to use `USR2` in place of `HUP` is because you are probably (should be) using `preload_app true` directive, specially in a Rails environment. The kill `USR2` works when your code was updated and the new process needs to get the fresh code. If you use `HUP` instead and `preload_app true` the code will not be updated.

## What is my point?

For me this `start` and `stop` actions mentioned in the example above are a dirty workaround. I don't like them. Sometimes my Monit doesn't work well and I can't blame it because I'm using a workaround. When Monit doesn't behave as expected, I usually do a manually combination of `pgrep` and `pkill`.

My suggestion is to use Monit as is.

```
check process unicorn_master with pidfile "…"
  start program = "/usr/bin/unicorn_start"
  stop program = "/usr/bin/unicorn_stop"
```

### Ok ok… How about graceful restart?

In most scenarios you only need graceful restart when:

1. Your application source code was updated. In this scenario, when you make a new deploy, instead of executing `monit restart name`, you can run the script `/usr/bin/unicorn_restart` in the end of your deploy. We use a [Capistrano](https://github.com/capistrano/capistrano) after deploy hook to execute the restart `run "pkill -USR2 -f unicorn_rails.*master"`.

2. A resource testing triggers a `restart`. In place of using `restart` action, try to execute a command `if cpu is greater than 50% for 5 cycles then exec "/usr/bin/unicorn_restart"`

### Resource testing

Beware of using Resource Testing.

Firstly, you can hide an internal problem. You don't need to be a hacker like the [Stripe guys](http://blog.nelhage.com/2013/03/tracking-an-eventmachine-leak/) going too deep to discover the cause of the memory leak, but you should, at least have a look at.

Secondly, if you are using [AWS AutoScaling](http://aws.amazon.com/autoscaling/), you probably use AutoScaling Policies ([avoid Cloud Smells](http://pablocantero.com/blog/2012/09/07/use-auto-scaling-avoid-cloud-smells/)). In this scenario your Resource Testing can conflict with AutoScaling Policies. If you have a Scale Up `if cpu > 80%` and a Resource Testing for the same, one will nullify the other.

