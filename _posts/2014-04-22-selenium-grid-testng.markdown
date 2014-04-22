---
layout: post
title:  "Testing Multiple Browsers"
date:   2014-04-22 12:00:00
categories: java
---

Web Applications normally need to be tested on multiple browsers.  We would like to write a single test suite and run it on different browsers.

We can provide the browser we want the test to run on with `@Parameters` annotation in TestNG.  In this case the defaul browser will be firefox.

{% highlight java %}
@Parameters({ "browser" })
@BeforeMethod
public void setUp(@Optional("firefox") String browser) throws Exception {
}
{% endhighlight %} 

Now we can configure TestNG with an xml file.

{% highlight java %}
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd" >
  
<suite name="SeleniumGrid" verbose="1" parallel="tests" thread-count="4">
  <test name="firefox">
    <parameter name="browser" value="firefox"/>
    <classes>
      <class name="jesg.SeleniumGridTest"/>
    </classes>
  </test>
  <test name="chrome">
    <parameter name="browser" value="chrome"/>
    <classes>
      <class name="jesg.SeleniumGridTest"/>
    </classes>
  </test>
  <test name="phantomjs">
    <parameter name="browser" value="phantomjs"/>
    <classes>
      <class name="jesg.SeleniumGridTest"/>
    </classes>
  </test>
</suite>
{% endhighlight %}

To run the test with a different browser we set the `browser` parameter.

When selenium server is running locally the test suite can be run with `mvn test`.

Full source:
{% highlight java %}
package jesg;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;

import org.openqa.selenium.Capabilities;
import org.openqa.selenium.HasCapabilities;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Optional;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;

public class SeleniumGridTest {
	private WebDriver driver;
	private Capabilities capabilities;
	private Map<String, Capabilities> browserConfig = new HashMap<String, Capabilities>();

	@BeforeTest
	public void init() {
		browserConfig.put("firefox", DesiredCapabilities.firefox());
		browserConfig.put("chrome", DesiredCapabilities.chrome());
		browserConfig.put("phantomjs", DesiredCapabilities.phantomjs());
	}

	@Parameters({ "browser" })
	@BeforeMethod
	public void setUp(@Optional("firefox") String browser) throws Exception {
		RemoteWebDriver remoteDriver = new RemoteWebDriver(
				browserConfig.get(browser));
		driver = remoteDriver;
		capabilities = ((HasCapabilities) remoteDriver).getCapabilities();
		driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
	}

	@AfterMethod
	public void tearDown() throws Exception {
		driver.quit();
	}

	@Test()
	public void verifyLinks() {
		driver.get("https://www.google.com/");
		System.out.println("Browser: " + capabilities.getBrowserName()
				+ " version: " + capabilities.getVersion() + 
				" platform: " + capabilities.getPlatform() + 
				" title: " + driver.getTitle());
	}

}
{% endhighlight %}

The complete project can be found on [github](https://github.com/jesg/selenium-grid-demo).