---
layout: post
title:  "Selenium - PhantomJS"
date:   2014-03-31 13:00:00
categories: selenium
---

[PhantomJS][phantomjs] is a headless WebKit.  It is good when running tests on a headless machine or when the tests need to be executed really fast.  In this example we will run Selenium WebDriver on [PhantomJS][phantomjs].

Install PhantomJS
-----------------
On `Arch Linux`
{% highlight text %}
$ pacman -S phantomjs
{% endhighlight %}

On `Debian Linux` in the unstable packages
{% highlight text %}
$ apt-get install phantomjs
{% endhighlight %}

For more information on installing PhantomJS refer to the [download](http://phantomjs.org/download.html) page.
Maven
-----

We need the following maven dependencies.

{% highlight xml %}
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.11</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>com.github.detro.ghostdriver</groupId>
	<artifactId>phantomjsdriver</artifactId>
	<version>1.1.0</version>
</dependency>
<dependency>
	<groupId>org.seleniumhq.selenium</groupId>
	<artifactId>selenium-java</artifactId>
	<version>2.41.0</version>
</dependency>
{% endhighlight %}

Test Case
---------
Finally we can write a test case that does a google search for `cheese` and `42`.

To improve performance we run PhantomJS in the background so we don't have to start and stop PhantomJS around each test.


{% highlight java %}
public class CheeseTest {
    
    private static PhantomJSDriverService PHANTOMJS_SERVICE;
    private WebDriver driver;
    
    @BeforeClass
    public static void before() throws IOException{
        PHANTOMJS_SERVICE = PhantomJSDriverService.createDefaultService();
        PHANTOMJS_SERVICE.start();
    }
    
    @AfterClass
    public static void cleanUp(){
        PHANTOMJS_SERVICE.stop();
    }

    @Before
    public void setUp() throws Exception {
        driver = new PhantomJSDriver(PHANTOMJS_SERVICE, DesiredCapabilities.phantomjs());
        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
    }

    @After
    public void tearDown() throws Exception {
        driver.quit();
    }

    @Test
    public void testCheese() {
        query("cheese");
    }
    
    @Test
    public void test42() {
        query("42");
    }
    
    private void query(String query) {
        driver.get("https://www.google.com");
        driver.findElement(By.name("q")).sendKeys(query);
        driver.findElement(By.name("btnG")).click();
        (new WebDriverWait(driver, 10)).until(ExpectedConditions
                .titleContains(query));
    }
}
{% endhighlight %}

The full project can be found on [github](https://github.com/jesg/selenium-phantomjs).

[phantomjs]: http://phantomjs.org/