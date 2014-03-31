---
layout: post
title:  "Selenium - Data Driven Testing"
date:   2014-03-31 12:00:00
categories: selenium
---

Data driven testing is a technique that drives tests with data.  Data driven tests externalize the test data in a file or in a database.

Web applications often have a large number of edge cases that need to be tested.  Fifty tests in a JUnit test case is hard to maintain.  By puting the test data in an external file it can be edited by developers, testers, and buisness analyst who may not be comfortable with a programming language.

Example
-----
Testing for broken links is a repetitive task that does not involve much logic.  When testing a link we need to know what page we start on, what link we click on, and the title of the next page rendered.  This can be summarized in a csv file.

{% highlight text %}
http://www.google.com/intl/en/about/products/,Web Search,Inside Search – Google
{% endhighlight %}

In this case we are testing the links on google's products page.

To read this data into our JUnit test we add `@RunWith(Parameterized.class)`.  A static method annotated with `@Parameters` provides the test data from an external source.  The `name` attribute in the `@Parameters` annotation defines the test name.  We the annotate the instance variables to be populated with `@Parameter`.  More information about `Parameterized` can be found [here](https://github.com/junit-team/junit/wiki/Parameterized-tests).

{% highlight java %}
@RunWith(Parameterized.class)
public class LinkTest {

    private WebDriver driver;
    private WebDriverWait wait;
    private static final Pattern COMMA = Pattern.compile(",");

    @Parameter(0)
    public String url;

    @Parameter(1)
    public String linkText;

    @Parameter(2)
    public String expectedTitle;

    @Parameters(name = "{index}: link[{1}] -> title[{2}]")
    public static Collection<String[]> data() throws IOException {
        List<String> lines = FileUtils.readLines(new File("links.csv"));
        List<String[]> csvData = new LinkedList<String[]>();
        for (String line : lines) {
            csvData.add(COMMA.split(line));
        }
        return csvData;
    }

    @Before
    public void setUp() throws Exception {
        driver = new FirefoxDriver();
        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
        wait = new WebDriverWait(driver, 10);
    }

    @After
    public void tearDown() throws Exception {
        driver.quit();
    }

    @Test
    public void testLink() {
        driver.get(url);
        driver.findElement(By.partialLinkText(linkText)).click();
        wait.until(ExpectedConditions.titleIs(expectedTitle));
        assertEquals(expectedTitle, driver.getTitle());
    }

}
{% endhighlight %}

The full project can be found on [github](https://github.com/jesg/selenium-ddt-links).
