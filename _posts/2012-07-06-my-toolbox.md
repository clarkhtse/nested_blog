---
layout: post
title: My toolbox
published: true
---

I haven't written here anything in a while. Whatever, I'd like to describe toolbox and setup.

### Hardware

First of all, most of magic happens on my Macbook Air 13". When I'm not on the go, or laying on the couch, the Macbook is hooked up with Thunderbolt Display.

![Macbook Air & Thunderbolt Display](http://f.cl.ly/items/1S3s330j0X001e0n2d14/Image%202012.08.06%204:46:01%20PM.png)

Macbook Air is really a beautiful, appealing thing. Not only it's a piece of hardware, but also the awesome experience. Small, thin, light, fast.

## OS

My OS of choice is, not hard to guess, OS X. Compared to Windows, its main pro is, it's the real **UNIX**, meaning I have my lovely terminal there, and all UNIX tools. (I can't even imagine myself working without that.)

A few pros over both Windows and different Linux distros:

* **UI/UX**, the one I really like it for. Stylish, elegant, Made For Humans™
* could be a part of previous, but I'd rather write it separately: **font rendering**. If you've ever seen how texts are rendered on Macs, I doubt you'll be able to look at how they are rendered on Windows/Linux machines without your eyes bleeding.
* nice **application ecosystem**

*(Shortly: I really do care about simplicily, power, and aethetics at the same time.)*

### Dev environment

My entire development environment can be bootstrapped on any Mac with single command:

    bash -c "$(curl -L https://raw.github.com/goshakkk/babushka-deps/master/install.sh)"
    
Brew for packages stuff.
    
I use Terminal.app and zsh as my shell, with [oh my zsh](https://github.com/robbyrussell/oh-my-zsh/). Vim is my editor of choice, with [janus](https://github.com/carlhuda/janus). (Here are my [dotfiles](https://github.com/goshakkk/dotfiles).)

My favorite coding/terminal color scheme is [Solarized](http://ethanschoonover.com/solarized). Choice of Light/Dark one depends on whenever the project I'm working on is personal or for client.

![VIM](http://f.cl.ly/items/3I1N3Q1q0x080x0P463W/Screen%20Shot%202012-08-06%20at%205.48.40%20PM.png)

Rack/Rails, node.js apps are run through [Pow](http://pow.cx) — while Pow supports the former natively, I'm using simple Pow port forwarding for the latter kind of apps.

I also run a [manservant](https://github.com/jimeh/manservant) instance using Pow on `man.dev`. With it, I'm able to look up manpages right from Safari.

![manservant](http://f.cl.ly/items/0B3T0F022e0e2T2R1k2x/Screen%20Shot%202012-08-06%20at%205.26.30%20PM.png)

Obviously, my personal & client apps are on Github. Github is especially useful for client app development — each repo has got a wiki for some project-related documents, Issues are awesome for both tracking bugs, offering new features, having discussions, and managing milestones.

### Technologies

For managing ruby versions & environments, I prefer [rbenv](https://github.com/sstephenson/rbenv/).

Most of times, I'm using the following stack:

* Ruby 1.9.3
* Rails 3
* [MongoDB](http://www.mongodb.org) for DB, and [Mongoid](http://mongoid.org/en/mongoid/index.html) as AR-like ORM
* [Redis](http://redis.io) with [redis-rb](https://github.com/redis/redis-rb)
* [Devise](https://github.com/plataformatec/devise) for authentification
* [CanCan](https://github.com/ryanb/cancan/) for ability management
* [HAML](http://haml.info)/[Slim](http://slim-lang.com) for templating
* [SCSS](http://sass-lang.com)/[Less](http://lesscss.org) for stylesheets
* [CoffeeScript](http://coffeescript.org) for JS
* [Twitter Bootstrap](http://twitter.github.com/bootstrap/) with some customization
* Old-fashioned, mostly static client side with some jQuery for animations/AJAX
* [RSpec](https://github.com/rspec/rspec/), [FactoryGirl](https://github.com/thoughtbot/factory_girl/) for testing (and [nyan cat rspec formatter](https://github.com/mattsears/nyan-cat-formatter))
* [Puma](http://puma.io)/[Thin](http://code.macournoyer.com/thin/) as web server

Whatsoever, I'm currently taking at [Backbone.js](http://backbonejs.org), [Underscore.js](http://underscorejs.org), [Chaplin](https://github.com/chaplinjs/chaplin), [Brunch](http://brunch.io), [CoffeeScript](http://coffeescript.org), [Less](http://lesscss.org), [Handlebars](http://handlebarsjs.com) for client side, which is a **simple, separate aplication**, depending only on backend app API.

Nevertheless, I don't like being locked into one language, I want to be polyglotic. I'm playing with a whole lot of different technologies in my spare time — [Node.js](http://nodejs.org), [Scala](http://www.scala-lang.org), [Erlang](http://www.erlang.org)/[Elixir](http://elixir-lang.org), [Clojure](http://clojure.org). (And whenever some of those make specific taks better than Ruby, I'm using it over Ruby. So in my recent client app I've got a zoo of a few apps — backend Rails app, utility Node.js application, communicating via REST API/[Pusher](http://pusher.com) with frontend Backbone app.)

When building apps, I follow the [Twelve-Factor App methodology](http://www.12factor.net).

### Deployment

I don't deploy my apps to separate own server, nor I really want to.

[Heroku](http://heroku.com) allows rapid, dead-simple deployment. And so, it's my the only deployment option (for now at least, I do understand that some very specific kinds of apps might require separate server). I like its simplicity (in deploying, scaling, managing env vars), I like lots of its hidden power ([buildpacks](https://devcenter.heroku.com/articles/buildpacks), for example). Deployed apps use [MongoLab](https://addons.heroku.com/mongolab) for MongoDB in the cloud, [Redis to go](https://addons.heroku.com/redistogo) for Redis.

I track exceptions with [Airbrake](https://addons.heroku.com/airbrake), store and browse heroku logs with [Papertrail](https://addons.heroku.com/papertrail).

### Stuff

Safari for browsing, [Tweetbot](http://tapbots.com/tweetbot_mac/) as Twitter client app, still Sparrow for email, iTunes for music, [Aperture](http://apple.com/aperture/) for photos, [1Password](https://agilebits.com/onepassword) for password management, [Reeder](http://reederapp.com/)/[Prismatic](http://getprismatic.com) for news, [Readability](http://www.readability.com/) for read-it-later thing, [Cloud](http://getcloudapp.com) app for screenshot sharing, [Flint](http://giantcomet.com/flint/) as [Campfire](http://campfirenow.com) client, [Grandview](http://www.darkheartfelt.com/grandview) for focused writing, [Divvy](http://mizage.com/divvy/) for window management, [Bartender](http://www.macbartender.com), [Eon](http://fuelcollective.com/eon) as client for [Freckle](http://letsfreckle.com), [OhLife](https://ohlife.com) for some sort of journal.

I also store my backups and photo library on 2Tb [Time Capsule](http://www.apple.com/timecapsule/).