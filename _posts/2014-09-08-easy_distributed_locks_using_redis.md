---
layout: post
title: "Easy distributed locks using Redis"
---

[Wombat](https://wombat.co) is a very concurrent application. It runs on multiple hosts, processes and threads, and sometimes we need to lock an execution to be exclusively performed across all threads, no matter the host or process. As any other distributed application, we can’t use a simple [Ruby Mutex](http://www.ruby-doc.org/core-2.1.1/Mutex.html), so we implemented a Mutex using Redis, which can be shared across all threads.

We decided to use [Redis](http://redis.io/), because it’s fast, consistent and has two super useful commands for locking: [SETNX](http://redis.io/commands/setnx) and [EXPIRE](http://redis.io/commands/setnx).

Read the full post: [Easy distributed locks using Redis](https://wombat.co/blog/easy_distributed_locks_using_redis/)