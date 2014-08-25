---
layout: post
title:  "Distributed Cucumber"
date:   2014-08-23 12:00:00
categories: testing
---

Distribute cucumber scenarios across multiple hosts.  Balance the load between
the hosts with a distributed queue on redis.

Setup on Arch Linux
---------

Install redis:
{% highlight text %}
$ sudo pacman -S redis
{% endhighlight %}

Start redis:
{% highlight text %}
$ redis-server
{% endhighlight %}

Make sure that ssh is setup.  Executing `ssh localhost` should log you into a
new session and not prompt for a password.

Setup the test
----------------

Write a simple feature file with tests steps.

When the test launches we need to add the scenarios to a redis list so the list is available before the workers startup on the remote hosts.  We will use simpleworker to distribute the tests.  Simpleworker will handle copying files, updating gems, and running a command on the remote host.

{% highlight ruby %}
# demo.rb

require 'redis'
require 'simpleworker'
require 'jcukeforker'

redis = Redis.new
key = "jcukeforker:#{redis.randomkey}"
redis.rpush key, JCukeForker::Scenarios.all
SimpleWorker::Runner.run "ruby launch.rb #{key}"
{% endhighlight %}

The remote hosts are setup in `simpleworker.yml`.  Change the user to your username.  You can find your username by running `whoami`.
{% highlight yaml %}
---
workers:
  - type: ssh
    directory: /tmp/foo
    user: jason
    host: localhost
{% endhighlight %}

Now we setup the workers that will execute scenarios from the redis
list.

{% highlight ruby %}
# launch.rb

require 'redis'
require 'jcukeforker'

class RedisScenarioQueue

  attr_reader :key

  def initialize(key)
    @redis = Redis.new
    @key = key
  end

  def shift
    @redis.lpop key
  end
end

queue = RedisScenarioQueue.new ARGV[0]

JCukeForker::Runner.run queue,
  :max => 2,
  :format => :pretty,
  :out => 'reports'
{% endhighlight %}

Jcukeforker (version >= 0.2.5) lazilly pops scenarios from the queue and distributes the task to a local worker.

To run the suite execute `ruby demo.rb`.

Better logging
--------------

When executing on muliple hosts it can be hard to tell which machine the output
corresponds to.  The following hack in sed will prepend user@hostname to each
line.

{% highlight ruby %}
SimpleWorker::Runner.run "ruby launch.rb #{key} | sed \"s/^/$(whoami)@$(hostname) /\""
{% endhighlight %}

The code for this example can be found on [github](https://github.com/jesg/distributed-cucumber-demo).

