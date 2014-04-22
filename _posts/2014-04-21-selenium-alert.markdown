---
layout: post
title:  "Selenium Alerts and Popups"
date:   2014-04-21 12:00:00
categories: java
---

Selenium 2.0 provides a class called Alert to handle popups.

Alert Example
------------

The following popup will greet the user with `Hello World!` popup when they click a button.

{% highlight xml %}

<form>
  <input name="alertButton" type="button" onclick="alert('Hello World!')" value="Greetings Alert"/>
</form>

{% endhighlight %}

<form>
  <input name="alertButton" type="button" onclick="alert('Hello World!')" value="Greetings Alert"/>
</form><br/>

In selenium we switch to the alert box:

{% highlight java %}
Alert alert = driver.switchTo().alert();
{% endhighlight %}

The alert class has the following methods:

-------------------------------------

1. accept()
2. dismiss()
3. getText()

-------------------------------------

The following tests the alert box on this page.

{% highlight java %}
package jesg;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.openqa.selenium.Alert;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;

public class AlertTest {
	private WebDriver driver;

	@Before
	public void setUp() throws Exception {
		driver = new FirefoxDriver();
	}

	@After
	public void tearDown() throws Exception {
		driver.quit();
	}

	@Test
	public void testAlert() {
		driver.get("http://jesg.github.io/java/2014/04/21/selenium-alert.html");
		driver.findElement(By.name("alertButton")).click();
		Alert alert = driver.switchTo().alert();
		System.out.println(alert.getText());
		alert.accept();

	}

}
{% endhighlight %}

The full source can be found on [github](https://github.com/jesg/selenium-alert).