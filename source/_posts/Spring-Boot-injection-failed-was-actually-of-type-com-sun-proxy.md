title: Spring Boot - Injection of resource dependencies failed, was actually of type 'com.sun.proxy.$Proxy
date: 2019-07-10 09:57:54
tags:
  - Java
  - Spring
  - Spring Boot
---

I've encountered a rather tricky bug while injecting a dependency in a Spring Boot application:

{% codeblock lang:bash %}
Error creating bean with name 'be.solidsyntax.ConsumingService': 
Injection of resourcedependencies failed; 
nested exception is org.springframework.beans.factory.BeanNotOfRequiredTypeException: 
Bean named 'dependency' is expected to be of type 'be.solidsyntax.Dependency'
 but was actually of type 'com.sun.proxy.$Proxy121'
at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
.postProcessProperties(CommonAnnotationBeanPostProcessor.java:324)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory
.populateBean(AbstractAutowireCapableBeanFactory.java:1411)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory
.autowireBeanProperties(AbstractAutowireCapableBeanFactory.java:391)
at org.springframework.test.context.support.DependencyInjectionTestExecutionListener
.injectDependencies(DependencyInjectionTestExecutionListener.java:119)
at org.springframework.test.context.support.DependencyInjectionTestExecutionListener
.prepareTestInstance(DependencyInjectionTestExecutionListener.java:83)
at org.springframework.boot.test.autoconfigure
.SpringBootDependencyInjectionTestExecutionListener
.prepareTestInstance(SpringBootDependencyInjectionTestExecutionListener.java:44)
{% endcodeblock %}

In most cases this exception is caused by injecting a bean which was enhanced by a Spring aspect into a field referencing an implementation instead on an interface. This was not the case in my setup.

<!-- more-->
### The setup

We have an Interface:
{% codeblock lang:java %}
    package be.solidsyntax;

    public interface Dependency {
    }
{% endcodeblock %}

A service implementing the interface:
{% codeblock lang:java %}
    package be.solidsyntax;

    import org.springframework.stereotype.Service;

    @Service("dependency")
    public class DependencyImpl implements Dependency {
    }
{% endcodeblock %}

And a service injecting the interface as a dependency:
{% codeblock lang:java %}
    package be.solidsyntax;

    import org.springframework.stereotype.Service;
    import javax.annotation.Resource;

    @Service
    public class ConsumingService {

        @Resource
        private Dependency dependency;

        public ConsumingService() {
        }
    }
{% endcodeblock %}

At no point are we trying to inject a concrete class (DependencyImpl) instead on an interface (Dependency), so this couldn't have been the problem.

### The solution

It turns out the problem lies with the *@Resource* annotation. There are some (subtle) differences between injecting dependencies with @Resource and @Autowired.
To quote the [Spring reference guide](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers):

{% blockquote %}
*@Resource* is semantically defined to identify a specific target component by its unique name, with the declared type being irrelevant for the matching process. *@Autowired* has rather different semantics: After selecting candidate beans by type, the specified String qualifier value is considered within those type-selected candidates only.
{% endblockquote %}

In short:
*@Resource* tries to inject a bean named 'dependency', which in this setup is of type DependencyImpl
*@Autowired* tries to inject a bean with type 'Dependency', of which we have one: DependencyImpl

Our fixed turns out to be rather simple for this case, replace *@Resource* with *@Autowired*:
{% codeblock lang:java %}
    package be.solidsyntax;

    import org.springframework.stereotype.Service;
    import javax.annotation.Resource;

    @Service
    public class ConsumingService {

        @Autowired
        private Dependency dependency;

        public ConsumingService() {
        }
    }
{% endcodeblock %}


