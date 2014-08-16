---
layout: post
title: "Mocha and subdirectories"
---

So, did you run ```mocha``` and your tests under subdirectories of ```test/``` weren't executed?

Don't worry, [you aren't alone](https://github.com/visionmedia/mocha/issues/106).

It's an expected behaviour of mocha.

[visionmedia.github.com/mocha/#best-practices](http://visionmedia.github.com/mocha/#best-practices)

> Best practices
> test/*
> By default mocha(1) will use the pattern ./test/*.js, so it’s usually a good place to put your tests.

The [recommended approach](http://visionmedia.github.com/mocha/#best-practices) is to use a Makefile (create one if you don't have it) and add your own test task.

    # Makefile
    TESTS = $(shell find test -name "*test.js")

    test:
      ./node_modules/.bin/mocha $(TESTS) --reporter list

    .PHONY: test

Then ```make test```.
