title: Clean the Spring Context between tests
tags:
  - Java
  - JUnit
  - Spring
id: 110
categories:
  - Lessons learned
date: 2013-10-22 14:19:07
---

When running a JUnit test with the [SpringJUnit4ClassRunner](http://docs.spring.io/spring/docs/3.2.x/javadoc-api/org/springframework/test/context/junit4/SpringJUnit4ClassRunner.html) the [Spring ](http://spring.io/)context is specified with the [ContextConfiguration ](http://docs.spring.io/spring/docs/3.2.x/javadoc-api/org/springframework/test/context/ContextConfiguration.html)annotation.

{% codeblock lang:java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("spring-context.xml")
public class MyTests {

    @Test
    public void aTest() {
    }
}
{% endcodeblock %}

Creating a Spring Context takes some time depending on the size of the application and the classpath that needs to be scanned. Therefore a Context will be stored in the Spring Context cache so that it can be reused for a different test-case.

Sometimes however your tests will change some values in the Spring Context which may cause other tests to fail. The solution is to clean the Spring Context between tests. This can be done with the [DirtiesContext ](http://docs.spring.io/spring/docs/3.2.x/javadoc-api/org/springframework/test/annotation/DirtiesContext.html)annotation.

To clean the Spring Context after the current test, place the DirtiesContext annotation on the method:
{% codeblock lang:java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("spring-context.xml")
public class MyTests {

    @Test
    @DirtiesContext
    public void aTest() {
    }
}   
{% endcodeblock %}

To clean the Spring Context after each test in the class, place the DirtiesContext annotation on the class and specify the ClassMode AFTER_EACH_TEST_METHOD:
{% codeblock lang:java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("spring-context.xml")
@DirtiesContext(classMode=ClassMode.AFTER_EACH_TEST_METHOD)
public class MyTests {

    @Test
    public void aTest() {
    }
}
{% endcodeblock %}

If no classMode is specified the default of AFTER_CLASS is used. Which will clean the Spring Context after running all tests in the class:
{% codeblock lang:java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("spring-context.xml")
@DirtiesContext
public class MyTests {

    @Test
    public void aTest() {
    }
}
{% endcodeblock %}