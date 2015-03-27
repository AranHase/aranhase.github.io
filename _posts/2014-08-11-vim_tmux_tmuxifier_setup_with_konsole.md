---
layout: post
title: '"vim + tmux + tmuxifier" setup with Konsole'
tags: vim
---

After trying many different terminal emulators to work with split panes and
facing all sort of problems I decided to give "vim + tmux + [tmuxifier][1]" a
try, and here is my setup.


Fixing wrong colors with vim
============================

The main issue I got with this setup is vim showing wrong colors. To fix this
problem I had to setup a few files. For the tmuxifier I had to setup
`~/.bashrc` as:

{% highlight bash %}
export PATH="${tmuxifier_dir}/bin:$PATH"
export TMUXIFIER_TMUX_OPTS="-2"
eval "$(tmuxifier init -)"
{% endhighlight %}


The `TMUXIFIER_TMUX_OPTS` var is important to force tmux to work with 256
colors. I also had to create a `~/.tmux.conf` file:

{% highlight bash %}
set -g default-terminal "screen-256color"
{% endhighlight %}


And that should fix the color issue.

Creating the window layout
==========================

When working on C++ projects, I like to use one window with three panes. One
for the code+vim, one to execute tests and show errors and one to execute the
application as a user. The layout is created using tmuxifier. So, my
`cppdev.window.sh` file (located at `${tmuxifier_dir}/layouts`) is:


{% highlight bash %}
# Set window root path. Default is `$session_root`.
# Must be called before `new_window`.
window_root "~/projects"

# Create new window. If no argument is given, window name will be based on
# layout file name.
new_window "cppdev"

# Split window into panes.
split_h 70
select_pane 1
split_v 30

# Set active pane.
select_pane 1
{% endhighlight %}


and a very simple session file named `cppdev.session.sh` (also located at
`${tmuxifier_dir}/layouts`) to launch the `cppdev` window created above:

{% highlight bash %}
# Create session with specified name if it does not already exist. If no
# argument is given, session name will be based on layout file name.
if initialize_session "Scppdev"; then
    # Load a defined window layout.
    load_window "cppdev"
    # Select the default active window on session creation. 
    select_window 1
fi

# Finalize session creation and switch/attach to it.
finalize_and_go_to_session 
{% endhighlight %}
    

To launch my C++ dev environment I just run:

{% highlight bash %}
$ tmuxifier load-session cppdev
{% endhighlight %}

Fixing `<CTRL+B>` conflict between tmux and vim
===============================================

tmux by default uses `<CTRL+B>` as the prefix key, which conflicts with the default vim shortcut to scroll a page backwards. There's an easy fix. Remap `<CTRL+B>` to `<CTRL+A>` in tmux. So, add these lines to your `~/.tmux.conf` file:

{% highlight bash %}
unbind C-b
set -g prefix C-a
{% endhighlight %}

My `~/.tmux.conf` with extra tweaks
===================================

{% highlight bash %}
unbind C-b
set -g prefix C-a

# Window numbers are always "gapless"
set-option -g renumber-windows on

set -g default-terminal "screen-256color"

# Smart pane switching with awareness of vim splits
# requires the vim-tmux-navigator plugin for vim (https://github.com/christoomey/vim-tmux-navigator)
bind -n C-h run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)g?(view|vim?)(diff)?$' && tmux send-keys C-h) || tmux select-pane -L"
bind -n C-j run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)g?(view|vim?)(diff)?$' && tmux send-keys C-j) || tmux select-pane -D"
bind -n C-k run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)g?(view|vim?)(diff)?$' && tmux send-keys C-k) || tmux select-pane -U"
bind -n C-l run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)g?(view|vim?)(diff)?$' && tmux send-keys C-l) || tmux select-pane -R"
bind -n C-\ run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)g?(view|vim?)(diff)?$' && tmux send-keys 'C-\\') || tmux select-pane -l"

# Default <ctrl+l> was to clean screen,
# now it is <prefix><ctrl+l>
bind C-l send-keys 'C-l'

# vi cursor moviment keys
set-window-option -g mode-keys vi

# start window,pane index at 1
set -g base-index 1
set -g pane-base-index 1

# stop tmux waiting for escape sequence
set -s escape-time 1

# move x clipboard into tmux paste buffer
bind C-p run "xclip -o -selection clipboard | tmux load-buffer -; tmux paste-buffer"

# copy selection vi-style
bind-key -t vi-copy 'v' begin-selection
bind-key -t vi-copy 'y' copy-selection

# auto copy tmux buffer to clipboard
bind -t vi-copy y copy-pipe "xclip -i -selection clipboard"
{% endhighlight %}

Useful links
============

*   [tmux cheatsheet][2]
*   [Fixing vim colors with tmux][3]
*   [tmuxifier github page][1]
*   [tmux manual][4]
*   [Seamlessly Navigate Vim and tmux Splits][5]
*   [Nichols`s tmux.conf file][6]
*   [Copy paste in tmux][7]
*   [SO: copy tmux buffer to clipboar][8]

 [1]: https://github.com/jimeh/tmuxifier
 [2]: https://gist.github.com/henrik/1967800
 [3]: http://stackoverflow.com/questions/10158508/lose-vim-colorscheme-in-tmux-mode
 [4]: http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man1/tmux.1?query=tmux&sec=1
 [5]: http://robots.thoughtbot.com/seamlessly-navigate-vim-and-tmux-splits
 [6]: http://zanshin.net/2013/09/05/my-tmux-configuration/
 [7]: http://awhan.wordpress.com/2010/06/20/copy-paste-in-tmux/
 [8]: http://unix.stackexchange.com/questions/15715/getting-tmux-to-copy-a-buffer-to-the-clipboard
