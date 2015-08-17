title: Access the concrete class from a spring proxy
tags:
  - AOP
  - Java
  - Spring
id: 194
categories:
  - Lessons learned
date: 2013-12-10 20:11:51
---

Working with [Spring-AOP](http://docs.spring.io/spring/docs/3.2.5.RELEASE/spring-framework-reference/html/aop.html) there are moments when you want to access the class type of the incoming method call. In my example I've implemented a [MethodInterceptor](http://aopalliance.sourceforge.net/doc/org/aopalliance/intercept/MethodInterceptor.html) to capture method invocations on a class of type net.solidsyntax.MyProxiedClass. 

{% codeblock lang:java %}
public Object invoke(MethodInvocation invocation) throws Throwable {
    Class<? extends Object> clazz = invocation.getThis().getClass()
    System.out.println(clazz);
}
{% endcodeblock %}

Surprisingly the output does not show something like:
class net.solidsyntax.MyProxiedClass

Instead we get a:
com.sun.proxy.$Proxy110

As it turns out the object on which we are working is already wrapped in a proxy by another aspect. In this case one to handle transactions.
So how do we access the concrete class from a spring proxy ?

Our first option is to use the Spring [AopUtils](http://docs.spring.io/spring/docs/3.0.x/api/org/springframework/aop/support/AopUtils.html).
{% codeblock lang:java %}
public Object invoke(MethodInvocation invocation) throws Throwable {
    Class<? extends Object> clazz = AopUtils.getTargetClass(invocation.getThis());
    System.out.println(clazz);
}
{% endcodeblock %}

However this solution still gives us:
com.sun.proxy.$Proxy110

The reason for this is rather simple. Not only is our object already wrapped by a transaction proxy, there is also a proxy for exception handling. We could iterate until we get a concrete class but we don't have to. Spring already contains [AopProxyUtils ](http://docs.spring.io/spring/docs/3.0.x/api/org/springframework/aop/framework/AopProxyUtils.html)which does the work for us. 

{% codeblock lang:java %}
public Object invoke(MethodInvocation invocation) throws Throwable {
    Class<?> clazz = AopProxyUtils.ultimateTargetClass(invocation.getThis());
    System.out.println(clazz);
}
{% endcodeblock %}

This changes our output to:
class net.solidsyntax.MyProxiedClass

AopProxyUtils can handle class-based proxies generated with CGLIB as well as interface proxies.