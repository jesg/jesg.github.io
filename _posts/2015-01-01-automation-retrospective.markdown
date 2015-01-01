---
layout: post
title:  "Retrospective Automation in 2014"
date:   2015-01-01 12:00:00
---

Here are my random thoughts on automation for 2014.

Ad Hoc Testing With Sikuli
--------------------------
Sikuli should only be used if you really need it.  Don't use sikuli if the
test can be done through service calls, web ui, database, or anything else
that does not require a graphical interface.  Testing with sikuli is brittle.

The following list will make your life miserable:

* fonts - The application may render fonts differently even when there are
slight changes to the underlying system.  Hence, each machine that runs the
test `must` be identical.
* packaging is limited to Debian Linux, so CentOS/RHEL fans have to create their own rpm's (yes there is more than one dependency (on CentOS/RHEL 6) and the documentation for building these dependencies is targeted to Debian). The situation is a little better on CentOS/RHEL 7.
* xvfb will not work.  Must run vncserver.
* no built in support for distributed execution.

I found that it was best to package the test environment in a docker image.
The image could be deployed via a private registry or a tar.
Using a DockerFile is an anti-pattern as it is impossible
to reproduce a docker image with a DockerFile.  Instead make commits on top
of your base image or role your own image with yum/debootstrap/chroot. In
retrospect I should have built my image with yum/chroot to reduce the image size.

Learn vncserver.  For security run vncserver on localhost.  Disable all tcp connections
if you don't need to connect to the session.  Recording the screen with ffmpeg
can reduce the time it takes to debug what went wrong.  I wrote a gem
[x11_recorder](https://github.com/jesg/x11_recorder) that wraps ffmpeg for X11 recording.

The current approach for remote execution with sikuli is to connect to a remote vnc session.
Leaving environment setup to the user.  I don't see any point in using
a remote vnc session if I have to setup the environment.  Though I could have key-bindings in openbox to
setup and tear down the environment.

Instead I wrote a gem [simpleworker](https://github.com/jesg/simpleworker)
to handle distributed execution.  Simpleworker copies the current workspace onto the remote hosts and
distributes tasks with a redis queue.  I use simpleworker to distribute cucumber scenarios accross
my sikuli VMs.


Test Data
---------
Be lazy.  Don't write test cases when a large portion
of the cases can be generated with a little prolog, ruby, and bash.


Local Test Environment
----------------------
To reduce friction with the test environment setup selenium grid locally on a docker image that mirrors the Linux distribution and
browser type and version CI runs against.  Run selenium
server on vncserver so you can view the tests without the browser stealing focus.
