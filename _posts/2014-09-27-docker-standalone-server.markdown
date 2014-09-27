---
layout: post
title:  "Selenium Standalone On Docker"
date:   2014-09-27 12:00:00
categories: testing
---

We will setup selenium standalone server on a docker image.

There are two major limitations to running automation tests locally.  Browsers
will steal focus preventing you from working on other stuff when the tests are
running.  Your browser version and OS are normally different from what the
selenium grid is running.  A docker image will allow us to run on the same OS
and browser version.  The tests will be viewed in vnc without the browser
stealing focus.

Setup
---------
Install [docker](https://docs.docker.com/installation) and start the docker
service.

Build the docker image from [github](https://github.com/jesg/docker-selenium-grid).
{% highlight text %}
git clone https://github.com/jesg/docker-selenium-grid.git
cd docker-selenium-grid/standalone-server
sudo docker build -t local/standalone-server .
{% endhighlight %}

Start Selenium Server
----------------

Run the docker image
{% highlight text %}
sudo docker run -p 4444:4444 -p 5910:5910 -t local/standalone-server
{% endhighlight %}

Install the `watir-webdriver` gem.
{% highlight text %}
gem install watir-webdriver
{% endhighlight %}

Open vncviewer to port 5910 (hint: password is 123456)
{% highlight text %}
vncviewer localhost::5910
{% endhighlight %}

Run a small ruby script that opens a browser
{% highlight ruby %}
require 'watir-webdriver'

browser = Watir::Browser.new(
  :remote, 
  :url => "http://127.0.0.1:4444/wd/hub",
  :desired_capabilities => Selenium::WebDriver::Remote::Capabilities.firefox)

browser.goto 'http://google.com'
puts browser.title
browser.quit
{% endhighlight %}

