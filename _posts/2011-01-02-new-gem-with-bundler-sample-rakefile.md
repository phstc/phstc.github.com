---
layout: post
title: New Gem with Bundler - Sample Rakefile
tags: [Bundler, English posts, Gem, Rakefile, Ruby]
---
{% include JB/setup %}

This post is an extension of the Ryan Bates post [New Gem with Bundler](http://railscasts.com/episodes/245-new-gem-with-bundler).

The idea of this post is only to demonstrate a very useful Rakefile to deploy and publish a gem.

If you want a step by step guide about the New Gem with Bundler, I recommend to check out the Ryan Bates [Railscasts](http://railscasts.com/episodes/245-new-gem-with-bundler) or [ASCIIcasts](http://asciicasts.com/episodes/245-new-gem-with-bundler). :)

The Rakefile demonstrated in this post is based on [MongoMapper Rakefile](https://github.com/jnunemaker/mongomapper/blob/master/Rakefile). I used this Rakefile when I created [mongomapper_id2](https://github.com/phstc/mongomapper_id2), It's a MongoMapper's plugin to add an auto incremented id to your MongoMapper's documents.

## Let's go, show me the code

Change the references of mongomapper_id2 to your gem's name.

[gist.github.com/762965](https://gist.github.com/762965)

    require 'rubygems'
    require 'rake'
    require 'rake/testtask'
    require File.expand_path('../lib/mongomapper_id2/version', __FILE__)

    Rake::TestTask.new(:test) do |test|
      test.libs << 'lib' << 'test'
      test.pattern = 'test/{functional,unit}/**/test_*.rb'
    end

    namespace :test do
      Rake::TestTask.new(:lint) do |test|
        test.libs << 'lib' << 'test'
        test.pattern = 'test/test_active_model_lint.rb'
      end

      task :all => ['test', 'test:lint']
    end

    task :default => 'test:all'

    desc 'Builds the gem'
    task :build do
      sh "gem build mongomapper_id2.gemspec"
    end

    desc 'Builds and installs the gem'
    task :install => :build do
      sh "gem install mongomapper_id2-#{MongomapperId2::VERSION}"
    end

    desc 'Tags version, pushes to remote, and pushes gem'
    task :release => :build do
      sh "git tag v#{MongomapperId2::VERSION}"
      sh "git push origin master"
      sh "git push origin v#{MongomapperId2::VERSION}"
      sh "gem push mongomapper_id2-#{MongomapperId2::VERSION}.gem"
    end

The purpose of the available tasks in this Rakefile are quite descriptive.

    rake test # to test
    rake build # to build
    rake install # to install the gem locally
    rake release # to release. explained bellow

##rake release

For me, the most interesting task is the `rake release`. This task creates a tag on GitHub with the version of your gem, push the locally committed files to the master on GitHub and publishes your gem on [RubyGems.org](http://rubygems.org). Remember, commit your changes, before run this task.

## Usage

Create your gem, run your tests.

    rake test

Build and install locally your gem.

    rake build
    rake install

Test your gem.

    # test.rb
    require 'my-gem-name'
    # code to execute your gem

If everything goes right… Publish your gem.

    git commit -a -m 'My commit'
    rake release
