---
layout: post
title:  "How To Start An Elixir App In Production"
date:   2015-09-12 12:56:29
categories: devops
---
Elixir has been gaining a lot of traction lately, particularly because of the immensely popular Phoenix web framework. Most of the users of Elixir tend to be smaller shops that have very simple deployment needs, and as such, documentation on large scale/automated deployment of Elixir apps is lacking.

### Commands
Elixir programs are run by executing a few different commands at the shell, not too dissimilar from Java, Erlang, etc. Elixir has several commands that do a few different things. The frustrating bit is all of these commands cross over and call each other in obscure and confusing ways. You'll see what I mean a bit later.

Below is an outline of the two commands we will be using. Note, these aren't all the commands but only the commands we need to productionize and support Elixir.

#### elixir
This command evaluates a project file and runs it, but also acts as an interface of sorts to the underlaying Erlang VM. This is ultimately the command that will run Erlang, and is what you will use to launch Elixir in production (sort of, keep reading).

#### mix
This command also launches projects and code, but is also the compile tool and the main tool you use to launch your app in production.

### Configuration
Elixir handles configuration in a way that is incredibly similar to Ruby/Rails. When starting your application, it will read from `config/config.exs`. At the bottom of that file, you should see something like so:

	import_config "#{Mix.env}.exs"

This tells Elixir to include another config file, ie: prod, dev, test. You can specify which environment to load by setting the `MIX_ENV` environmental variable when starting your application. For example, setting:

	MIX_ENV=prod

will load `config/prod.exs`.

### Compiling
Compiling is a one liner:

	mix compile

while in your project root. If you like to be explicit, you can do:

	mix deps.compile
	mix compile

Which will compile deps independently.

### Running
Running an Elixir app is a bit tricky depending on your needs. If you're running distributed nodes, you must run it like so:

	MIX_ENV=prod elixir --name app@hostname --cookie "MyErlangCookie" -S mix run --no-compile --no-halt

A few things to note:

1. You should be using an init system that is event based. That is, use Upstart, systemd, runit, etc. Do not use sysvinit.
2. `elixir` sets Erlang parameters, and in this case we're setting the node name and Erlang cookie.
3. Replace the `app@hostname` bit with something more suitable for your environment.
4. Passing `-S mix run` tells Elixir to start the app it self using mix. Since mix is compile tool, we want to instruct mix not to compile our code in production, and to not halt once the code as started running.
5. The `MIX_ENV` can be omitted and instead put into some other facility of your init script, ie: Upstart env stanza, etc.

Using the above command, your Elixir app will launch cleanly into the foreground... unless you are using Phoenix! There is one last thing that you must do in order to make a Phoenix app work. Edit your `config/prod.exs` file and make sure the following is uncommented:

	config :phoenix, :serve_endpoints, true

This will start Phoenix when you do `mix run`. Without the above line, Phoenix won't be told to start up, resulting in all your code launching, except the web server it self.

That's about it!

### TL;DR
Compile like this:

	mix deps.compile
	mix compile

If using Phoenix, add this to the following to `config/prod.exs`

	config :phoenix, :serve_endpoints, true

then start your app using Upstart, systemd, runit, and let it handle the **foreground** process like so

	MIX_ENV=prod elixir --name app@hostname --cookie "MyErlangCookie" -S mix run --no-compile --no-halt

Enjoy!