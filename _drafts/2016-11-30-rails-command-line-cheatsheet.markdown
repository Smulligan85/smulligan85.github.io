---
layout: post
title: "Rails Command Line Cheatsheet"
date:   2016-11-30
categories: tutorial
---

I thought it would be helpful to create a quick cheatsheet of some of the most useful commands that can be run on the command line to increase productivity when working with Rails. Let's get started!

The `rails new` command will set up a new Rails project directory.

{% highlight bash %}
$ rails new commandlinecheatsheet
{% endhighlight %}

You can also optionally specify the control system and database preference from the beginning with `rails new`.

{% highlight bash %}
$ mkdir commandlinecheatsheet
$ cd commandlinecheatsheet
$ git init
Initialized empty Git repository in .git/
$ rails new . --git --database=postgresql
{% endhighlight %}

Another useful command is the Rails `generate` or `g` command. `generate` can be used to create a wide variety of boilerplate code. Let's look at creating controllers with `generate`.

{% highlight bash %}
$ rails g CommandCheats new generate migrate
{% endhighlight %}

In addition to creating the `command_cheats_controller.rb` file with a `new` `generate` and `migrate` action defined, this command will also create various erb, testing and coffeescript files. This presents one of the possible downsides of using generators, since you may not want some or all of the these extraneous files. Alternatively, you can specify which types of files you would like the generator to skip.

{% highlight bash %}
$ rails g CommandCheats new generate migrate --skip-assets --skip-helper --skip-controller-specs --view-specs
{% endhighlight %}

Another useful command is the `rails console` or `rails c`. This opens a `irb` environment that allows you to play around with ideas or make changes to data within your terminal. This command is really nice, but I usually use it in tandem with the flag `--sandbox`. The command `rails c --sandbox` opens up the same `irb` environment, but will rollback any changes you make to the db after you exit out. I believe this is a better way of testing out ideas, without having to worry about unintentionally altering your application.  

I hope that you found this post helpful as a quick reference to some common Rails CLI commands. Feel free to reach out on Twitter with any questions.



