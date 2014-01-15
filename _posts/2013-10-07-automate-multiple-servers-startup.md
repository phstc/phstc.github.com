---
layout: post
title: "Automate multiple servers startup"
description: ""
category:
tags: [Foreman, Tmuxinator]
---
{% include JB/setup %}

So, do you need to startup multiple servers or services to start your development environment? I do!

We can split the Terminal in panels and start each server manually, it will work, but when we need to do it every day, it becomes boring.

There are many ways to automate this startup, but the two that I like most are with [Tmuxinator](https://github.com/aziz/tmuxinator) or with [Foreman](https://github.com/ddollar/foreman).

## Tmuxinator

Create a `~/.tmuxinator/development.yml`.

    # ~/.tmuxinator/development.yml
    project_name: development
    project_root: /Users/pablo/workspace/
    windows:
      - servers:
          layout: even-vertical
          panes:
            - mongod
            - cd /Users/pablo/workspace/project1 && foreman start
            - cd /Users/pablo/workspace/project2 && rails s -p 4000
            - cd /Users/pablo/workspace/project3 && TOKEN=123 rackup
            - cd /Users/pablo/workspace/sandbox && rails s

Then run:

    $ mux development

## Foreman

Create a `Procfile_development`.

    # ~/Procfile_development
    mongo: mongod
    project1: sh -c 'cd /Users/pablo/workspace/project1 && foreman start'
    project2: sh -c 'cd /Users/pablo/workspace/project2 && rails s -p 4000'
    project3: sh -c 'cd /Users/pablo/workspace/project3 && TOKEN=123 rackup'
    sandbox: sh -c 'cd /Users/pablo/workspace/sandbox && rails s'

Then run:

    $ foreman start -f ~/Procfile_development