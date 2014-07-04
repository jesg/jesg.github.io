---
layout: post
title:  "Headless Sikuli with VNC"
date:   2014-07-04 12:00:00
categories: testing
---

VNC can be used to run sikuli scripts on headless machines.  We'd like to
launch a script on a vncserver without opening vncviewer.

Setup on Arch Linux
---------

First install some dependencies for sikuli and vnc.

{% highlight text %}
$ sudo pacman -S tigervnc gcc opencv tesseract jdk7-openjdk
{% endhighlight %}

Download [sikuli 1.0.1](https://launchpad.net/sikuli/+download) and launch the
installer
{% highlight text %}
$ java -jar sikuli-setup.jar
{% endhighlight %}

Select the 4th option.  Set the SIKULIX\_HOME environmental variable to the
path to sikuli-java.jar.

Install rvm and jruby:
{% highlight text %}
$ \curl -sSL https://get.rvm.io | bash -s stable
$ rvm install jruby
{% endhighlight %}

Start VNC Server
----------------

{% highlight text %}
$ vncserver
{% endhighlight %}

Edit `~/.vnc/xstartup` to modify the graphical environment.  You should install
a minimal stacking window manager like openbox, twm, or fluxbox.

Create a file `sikuli-example.rb` with the following contents
{% highlight ruby %}
require 'watir-webdriver'
require 'rukuli'

browser = Watir::Browser.start 'www.google.com'
browser.text_field(:name => 'q').set ''
puts browser.title

screen = Rukuli::Screen.new
screen.type 'yellow'
browser.screenshot.save 'google.png'
browser.close
{% endhighlight %}

This will startup a firefox browser and sikuli will type 'yellow' into the
search region.

We need to install watir and rukili.
{% highlight text %}
$ rvm use jruby
$ gem install watir-webdriver
$ gem install rukuli
{% endhighlight %}

Run Script on VNC
-----------------

Now we can run our script in vnc. Change the display to match the display you
saw when you ran `vncserver`.
{% highlight text %}
$ xterm -display :1 -e jruby sikuli-example.rb
{% endhighlight %}

The script should take a picture of google's home page after sikuli types in
`yellow`.

You can open `google.png` in your favorite picture viewer.
