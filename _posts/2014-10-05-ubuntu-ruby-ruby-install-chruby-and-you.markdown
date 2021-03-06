---
wordpress_id: RB-358
layout: post
title: Ubuntu, Ruby, ruby-install, chruby, Rails and You
ruby_version: 2.6.5
rails_version: 6.0.0
ruby_install_version: 0.7.0
chruby_version: 0.3.9
---

**Last updated: October 9th 2019**


**This beginner's guide will set your machine up with Ruby {{page.ruby_version}} using chruby+ruby-install and Rails {{page.rails_version}} and is specifically written for a _development_ environment on Ubuntu 19.04, but will probably work on many other operating systems, including older / newer versions of Ubuntu and Debian. YMMV.**

<div class="warning">
  Under no circumstance should you install Ruby, Rubygems or any Ruby-related packages from apt-get. This system is out-dated and leads to major headaches. Avoid it for Ruby-related packages. We do Ruby, we know what's best. Trust us.
</div>

This guide will cover installing a couple of things:

* [**ruby-install**](https://github.com/postmodern/ruby-install): a very lightweight way to install multiple Rubies on the same box.
* [**chruby**](https://github.com/postmodern/chruby): a way to easily switch between those Ruby installs
* **Ruby {{page.ruby_version}}**: at the time of writing the newest current stable release of Ruby.
* **Bundler**: a package dependency manager used in the Ruby community
* **Rails {{page.rails_version}}**: at the time of writing the newest current stable release of Rails.

By the end of this guide, you will have these things installed and have some very, very easy ways to manage gem dependencies for your different applications / libraries, as well as having multiple Ruby versions installed and usable all at once.

We assume you have `sudo` access to your machine, and that you have an understanding of the basic concepts of Ruby, such as "What is RubyGems?" and more importantly "How do I turn this computer-thing on?". This knowledge can be garnered by reading the first chapter of [any Ruby book](https://manning.com/black2).

### Housekeeping

First of all, we're going to run `sudo apt-get update` so that we have the latest sources on our box so that we don't run into any package-related issues, such as not being able to install some packages.

Next, we'll run another command which will install the essential building tools that will be used to install Ruby:

```
sudo apt-get install build-essential
```

And now we're ready to install ruby-install.

### ruby-install

The installation instructions can be found [on the README of ruby-install](https://github.com/postmodern/ruby-install#install), but I'll repeat them here so you don't have to go over there:

```
wget -O ruby-install-{{page.ruby_install_version}}.tar.gz \
  https://github.com/postmodern/ruby-install/archive/v{{page.ruby_install_version}}.tar.gz
tar -xzvf ruby-install-{{page.ruby_install_version}}.tar.gz
cd ruby-install-{{page.ruby_install_version}}/
sudo make install
```

First we fetch the ruby-install file, extract it into a directory, then make it. You can verify that these steps have worked by running the following command:

```
$ ruby-install -V
```

If you see this, then you've successfully installed ruby-install:

```
ruby-install: {{page.ruby_install_version}}
```

### Ruby

Our next step is to install Ruby itself, which we can do with this command:

```
ruby-install ruby {{page.ruby_version}}
```

This command will take a couple of minutes, so grab your $DRINKOFCHOICE and go outside or something. Once it's done, we'll have Ruby {{page.ruby_version}} installed. In order to use this Ruby version, we'll need to install chruby as well. The instructions [can be found in chruby's README](https://github.com/postmodern/chruby#install) too, but I will reproduce them here:

```
wget -O chruby-{{page.chruby_version}}.tar.gz \
  https://github.com/postmodern/chruby/archive/v{{page.chruby_version}}.tar.gz
tar -xzvf chruby-{{page.chruby_version}}.tar.gz
cd chruby-{{page.chruby_version}}/
sudo make install
```

After this has been installed, we'll need to load chruby automatically, which we can do by adding these lines to your shells configuration file using the following command:

```
cat >> ~/.$(basename $SHELL)rc <<EOF
source /usr/local/share/chruby/chruby.sh
source /usr/local/share/chruby/auto.sh
EOF
```

In order for this to take effect, we'll reload the shell

```
exec $SHELL
```

Alternatively, opening a new terminal tab/window will do the same thing.

To verify that chruby is installed and has detected our Ruby installation, run `chruby`. If you see this, then it's working:

```
ruby-{{page.ruby_version}}
```

Now we need to make that Ruby the default Ruby for our system, which we can do by creating a new file called `~/.ruby-version` with this content:

```
ruby-{{page.ruby_version}}
```

This file tells `chruby` which Ruby we want to use by default. To change the ruby version that we're using, we can run `chruby ruby-{{page.ruby_version}}` for example -- assuming that we have Ruby {{page.ruby_version}} installed first!

Did this work? Let's find out by running `ruby -v`:

```
ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-linux]
```

### Rails

Now that we have a version of Ruby installed, we can install Rails. Because our Ruby is installed to our home directory, we don't need to use that nasty `sudo` to install things; we've got write-access! To install the Rails gem we'll run this command:

    gem install rails -v {{page.rails_version}} --no-rdoc --no-ri

This will install the `rails` gem and the multitude of gems that it and its dependencies depend on, including Bundler.

### Rails pre-requisites

Before we can start a new Rails app, there are a few more things that we need to install.


#### JavaScript Runtime

Rails requires a JavaScript runtime to run Webpacker for its assets.

To fix this error install `nodejs`, which comes with a JavaScript runtime:

```
sudo apt-get install nodejs
```

On top of this, modern Rails uses `yarn`, a JavaScript package manager. We will need to install that too. The [instructions are on the Yarn site](https://yarnpkg.com/lang/en/docs/install/#debian-stable) but here they are too:

```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
```

#### SQLite3

First of all, we need to install `libsqlite3-dev`, which is the package for the development headers for SQLite3. We need this so that when `bundle install` runs as a part of `rails new`, it will be able to install the `sqlite3` gem that is a default dependency of Rails applications.

(If you're using MySQL or PostgreSQL instead, see the sections below.)

#### MySQL

If you're planning on using the `mysql2` gem for your application then you'll want to install the `libmysqlclient-dev` package before you do that. Without it, you'll get an error when the gem tries to compile its native extensions:

    Building native extensions.  This could take a while...
    ERROR:  Error installing mysql2:
        ERROR: Failed to build gem native extension.

        /home/ryan/.rubies/ruby-2.3.0/bin/ruby extconf.rb
    checking for ruby/thread.h... yes
    checking for rb_thread_call_without_gvl() in ruby/thread.h... yes
    checking for rb_thread_blocking_region()... yes
    checking for rb_wait_for_single_fd()... yes
    checking for rb_hash_dup()... yes
    checking for rb_intern3()... yes
    checking for mysql_query() in -lmysqlclient... no
    checking for main() in -lm... yes
    checking for mysql_query() in -lmysqlclient... no
    checking for main() in -lz... yes
    checking for mysql_query() in -lmysqlclient... no
    checking for main() in -lsocket... no
    checking for mysql_query() in -lmysqlclient... no
    checking for main() in -lnsl... yes
    checking for mysql_query() in -lmysqlclient... no
    checking for main() in -lmygcc... no
    checking for mysql_query() in -lmysqlclient... no
    *** extconf.rb failed ***
    Could not create Makefile due to some reason, probably lack of necessary
    libraries and/or headers.  Check the mkmf.log file for more details.  You may
    need configuration options.

Install this package using `sudo apt-get install libmysqlclient-dev` and then the `mysql2` gem will install fine.

#### PostgreSQL

Similar to the `mysql2` gem's error above, you'll also get an error with the `pg` gem if you don't have the `libpq-dev` package installed you'll get this error:

    Building native extensions.  This could take a while...
    ERROR:  Error installing pg:
        ERROR: Failed to build gem native extension.

        /home/ryan/.rubies/ruby-2.3.0/bin/ruby extconf.rb
    checking for pg_config... no
    No pg_config... trying anyway. If building fails, please try again with
     --with-pg-config=/path/to/pg_config
    checking for libpq-fe.h... no
    Can't find the 'libpq-fe.h header
    *** extconf.rb failed ***
    Could not create Makefile due to some reason, probably lack of necessary
    libraries and/or headers.  Check the mkmf.log file for more details.  You may
    need configuration options.

Install this package using `sudo apt-get install libpq-dev`.

### Fin

And that's it! Now you've got a Ruby environment you can use to write your (first?) Rails application in with such minimal effort. A good read after this would be the <a href='http://guides.rubyonrails.org'>official guides for Ruby on Rails</a>.

The combination of chruby and ruby-install is such a powerful tool and comes in handy for day-to-day Ruby development. Use it, and not the packages from apt to live a life of development luxury.
