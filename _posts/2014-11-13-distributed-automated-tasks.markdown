---
layout: post
title:  "Distibuted Automated Testing"
date:   2014-11-13 12:00:00
categories: testing
---

There comes a time as an Automation Engineer that you need to distribute tasks across multiple
machines.  This normally happens because your not doing web testing so you cannot use selenium
grid but you still want to distribute the work load.  The traditional[1](https://github.com/joakimk/testbot)
way of doing this is to split up large chunks of work like feature files and run them on a remote hosts.

There are two problems with this approach.  First the unit of work is to large so some machines will finish
sooner than others.  The second is that stuff breaks when you distribute work to multiple machines.  The
traditional fire and forget technique does not detect failure.

Why not use [resque](https://github.com/resque/resque) to manage background workers?  The problem
with resque is that it does not give a clean solution for distributing files to remote nodes.  The other more
serious problem with resque is that there is no clean way to load a fresh environment on the remote node.

So what I want to build with [simpleworker](https://github.com/jesg/simpleworker) is a framework for fine grain reliable test execution on a collection of remote hosts.

Example
-------
Nothing explains a framework like an example.  This example can be found on [github](https://github.com/jesg/simpleworker/blob/master/examples/basic.rb).
Other examples can be found in the [examples](https://github.com/jesg/simpleworker/tree/master/examples) directory.

```ruby
require 'simpleworker'

tasks = ['first', 'second', 'third']
redis = Redis.new
runner = SimpleWorker::Runner.new(redis, tasks,
                                  :max_retries => 2)

worker_thread = Thread.new do
  task_queue = SimpleWorker::TaskQueue.new(redis, 'my_hostname', runner.jobid)

  task_queue.fire_start

  task_queue.each_task do |task|
    if task == 'first'
      sleep 15
    elsif task == 'second'
      task_queue.expire_current_task task
    else
      task_queue.fire_log_message "Task: #{task}"
    end
  end

  task_queue.fire_stop
end

runner.run
```

So let's break the program down. The first thing we do is create an array
of tasks.
Next, we create a connection to a redis server we are running locally.  After
which we create the runner who will manage the tasks.  On creation the
runner will add tasks to the queue on redis.  When a task has
expired the runner can attempt to retry up to 2 times. Tasks that expire
at the very end of the job may or may not be retried.  This is to ensure
the workers do not hang at the end of the test.  Shutting down
a worker has higher priority than retrying an expired task.
If `:max_retries`
is not set simpleworker will only try to execute a task once and log all
expired tasks.

```ruby
runner = SimpleWorker::Runner.new(redis, tasks,
                                  :max_retries => 2)
```

Now we create a thread that will work on the tasks in the redis queue.
Look in the examples for how to create a local subprocess and how to
create a remote process.  We invoke `fire_start` and `fire_stop` to signal
the runner that a worker is starting and stopping respectively.  The core
of the program is where we iterate through each task.

```ruby
task_queue.each_task do |task|
  if task == 'first'
    sleep 15
  elsif task == 'second'
    task_queue.expire_current_task task
  else
    task_queue.fire_log_message "Task: #{task}"
  end
end
```
By default the task is set to expire after 10 seconds on redis.
This timout can be configured with the runner's `:task_timeout` option.
We can schedule to retry a task with `expire_current_task`.  The runner will
ignore the request if the task has already been attempted `:max_retries`.

Use Cases
---------
Here I will wildly speculate where [simpleworker](https://github.com/jesg/simpleworker)
may be a good fit.

* non web based integration testing
* resource intense integration testing
* web based testing when selenium grid faces significant lag
* linux test machines

And here are some places that I don't think [simpleworker](https://github.com/jesg/simpleworker)
is a good fit.

* web based integration testing
* fast test suite that does not take much memory or network
* anything that starts up a database or web application in the test
* windows (feel free to submit a patch for windows support)
