---
layout: post
title:  "TestNG - Hello World"
date:   2014-03-28 12:00:00
categories: test
---

An example to get started with TestNG and maven.

Maven
-----

Add the following to the maven dependencies.

{% highlight xml %}
<dependency>
  <groupId>org.testng</groupId>
  <artifactId>testng</artifactId>
  <version>6.8.8</version>
  <scope>test</scope>
</dependency>
{% endhighlight %}

Create a Test
-------------

Next create a class in `src/test/java` called `ExampleTest`.

{% highlight java %}
import org.testng.annotations.Test;
import static org.testng.Assert.*;

public class ExampleTest {
    
    @Test
    public void exampleTest(){
        assertEquals(2 + 2, 4, "2 + 2 should equal 4");
    }

}
{% endhighlight %}

Run the Test
------------
Execute `mvn test` in the maven project to run the test.

The code for this example can be found on [github](https://github.com/jesg/testng-hello-world).