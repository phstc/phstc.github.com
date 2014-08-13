---
layout: post
title: Automate multiple servers startup
---

So, do you need to startup multiple servers or services to start your development environment? I do!

We can split the Terminal in panels and start each server manually, it will work, but when we need to do it every day, it becomes boring.

There are many ways to automate this startup, but the two that I like most are with [Tmuxinator](https://github.com/aziz/tmuxinator) or with [Foreman](https://github.com/ddollar/foreman).

I usually use more Tmuxinator than Foreman for this automation, as I use [Tmux](http://tmux.sourceforge.net/) for my development environment. But both work fine and also work as a documentation. If someone wants to run an app that depends on others, you can send the Tmuxinator project file or Procfile to describe what's required to get the app up and running.

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
            - cd /Users/pablo/workspace/sandbox  && rails s


Then run:

    $ mux development

### Tips

#### Tmux Status Line

I show the `tmuxinator project name/pane title:window number` in [my left status line](https://github.com/phstc/dotfiles/blob/master/tmux.conf#L93-L110). The pane title is very useful when you have multiples panes in the same window.

    # tmux.conf
    set -g status-utf8 on

    # status line left side
    set -g status-left-length 40
    set -g status-left "#[bold,fg=green]#S/#T:#[fg=yellow]#I"

    # set -g status-left-length 100
    # set -g status-left '#[fg=green] #T#[default]'

    # status line right side
    # 15% | 28 Nov 08:15AM
    set -g status-right "#[bg=colour251,fg=colour240]%I:%M%p #[bg=colour240,fg=colour251] %d %b"

    # update the status bar every sixty seconds
    set -g status-interval 30

    # center the window list
    set -g status-justify centre

To define the pane title I have this function in my [~/.bashrc](https://github.com/phstc/dotfiles/blob/master/bashrc#L71-L73):

    # ~/.bashrc
    func set_tmux_pane_title() {
      printf "\033]2;%s\033\\" "$*";
    }

Then in the Tmuxinator project file I set the pane title as follows:

    # ~/.tmuxinator/development.yml
    project_name: development
    project_root: /Users/pablo/workspace/
    windows:
      - servers:
          layout: even-vertical
          panes:
            - set_tmux_pane_title mongo    && mongod
            - set_tmux_pane_title project1 && cd /Users/pablo/workspace/project1 && foreman start
            - set_tmux_pane_title project2 && cd /Users/pablo/workspace/project2 && rails s -p 4000
            - set_tmux_pane_title project3 && cd /Users/pablo/workspace/project3 && TOKEN=123 rackup
            - set_tmux_pane_title sandbox  && cd /Users/pablo/workspace/sandbox  && rails s

#### Switch panes

To [switch panes (up/down/left/right)](https://github.com/phstc/dotfiles/blob/master/tmux.conf#L57-L61) I use `ctrl+shift+arrow keys`:

    # ~/.tmux.conf
    bind -n S-C-Up select-pane -U
    bind -n S-C-Down select-pane -D
    bind -n S-C-Left select-pane -L
    bind -n S-C-Right select-pane -R

#### Zoom in/Zoom out

* `Ctrl + b + z` - zoom in
* `Ctrl + b + z` - zoom out


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